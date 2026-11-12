# Шаблон: стек на сервере, reverse-proxy с HTTPS и скрипт деплоя (практика №7)

Тексты файлов папки `deploy/` для [практики №7](practice-7-guide.md): `compose.yaml` стека на сервере, конфигурация reverse-proxy (Caddy, Nginx или Traefik — по п. 1 «Решения» ADR 006), `.env.example`, скрипт деплоя `deploy.sh` и обвязка способов автодеплоя B и C (п. 4). Какой раздел брать, говорит шаг инструкции; здесь — только тексты и то, что в них менять.

Как пользоваться:

- Файл копируется из блока целиком, затем меняются строки с пометкой `АДАПТИРОВАТЬ`. Когда строка приведена к вашему значению, пометку удалите — комментарий в конце строки или строку-комментарий `# АДАПТИРОВАТЬ: ...` целиком; остальные комментарии оставьте. Проверка — шаг 1.4 инструкции: `git grep` по слову пометки в `deploy/` пуст
- Значения сервера — поддомен, имя образа, тег, пароль базы — только в `deploy/.env` на сервере. В git — `deploy/.env.example` без настоящих значений
- Теги образов закреплены: `caddy:2.11.9`, `nginx:1.30.6`, `traefik:v3.7.18`, `certbot/certbot:v5.10.0`; `postgres` — та же версия, что в вашем `compose.yaml`. Плавающий тег (`caddy:2`) при следующем `docker compose pull` может принести другую версию с другим поведением

**Что проверено при подготовке (2026-11-11).** Локально, Windows 11, Docker 29.8.1, Compose 5.5.1, образ приложения — проект после практики №6 (uv, Python 3.14, fastapi 0.143.0, uvicorn 0.54.0) с логами по [шаблону логирования](logging-template.md); адрес сайта `localhost`, поэтому вместо Let's Encrypt — самоподписанный или внутренний сертификат.

| Что | Результат |
|---|---|
| `docker compose config` для трёх вариантов прокси | Без ошибок; без `deploy/.env` — ошибки `required variable ... is missing a value` по каждой переменной |
| Caddy 2.11.9 | `http://` → `308 Permanent Redirect` на `https://`; `/health` через прокси `200`, заголовок `X-Request-ID` проходит в обе стороны |
| Traefik v3.7.18 с Docker 29.8.1 | Находит `app` по меткам; `http://` → `308`; `/health` `200`, `X-Request-ID` проходит. Запрос к Let's Encrypt без `email` отправлен (отказ только из-за имени `localhost`) |
| Nginx 1.30.6 | `http://` → `301`; `/health` `200`, `X-Request-ID` проходит; после пересоздания `app` с новым IP — снова `200` |
| `certbot certonly --standalone` через `docker compose run` | Сервис с `profiles` запускается без `--profile`, порт 80 занимает, до Let's Encrypt доходит |
| `deploy.sh` | Тег в `.env` меняется, `up -d --wait` дожидается `healthy`, неверный аргумент — код 2; `curl --resolve` к прокси на `127.0.0.1` — `200`. С временным сертификатом Caddy (`tls internal`): без `CURL_INSECURE=1` — `curl: (60)`, `000`, код 1; с ним — `200`, код 0. Повтор проверки `/health` (2026-11-12, Traefik v3.7.18, `TRAEFIK DEFAULT CERT` с `CURL_INSECURE=1`): прокси поднят через 20 с после старта скрипта — `200`, код 0; прокси остановлен — около 30 с попыток, одна строка `curl: (7)`, `000`, код 1. На имитации сервера (Traefik, способ C): два деплоя по `cron` с пересозданием `app` — `200`, код 0. Проверен `shellcheck` |
| `up -d --wait app` при сервисе `migrate` (`service_completed_successfully`, раздел 7) | Compose 5.5.1: первый запуск и смена тега — `migrate` `Exited (0)`, `app` `Healthy`, код 0 |
| Временный сертификат Nginx (раздел 2.2) | Команда `openssl` в контейнере certbot создаёт файлы в томе; Nginx стартует, `curl -k` — ответ, без `-k` — ошибка сертификата; удаление временного — командой блока. С тестовым ACME-сервером (Pebble), certbot v5.10.0: после удаления — выпуск в `live/<имя>`; без удаления — `live directory exists for <имя>`, после команды удаления — выпуск в `live/<имя>` |
| Опция `restrict` ключа деплоя, `cron` и `flock` | В контейнере Ubuntu 24.04 (OpenSSH 9.6p1, util-linux 2.39.3): по ключу с `restrict` команда джоба выполняется, терминал и проброс портов запрещены; строка crontab ставится и снимается командами раздела 6; при занятом замке второй запуск `flock -n` выходит без деплоя |
| Джоб `deploy` (способ B) | `actionlint` 1.7.12 — без замечаний |
| `poll.sh` (способ C) | Только `shellcheck` |
| Перезапуск | `kill -TERM 1` в `app` → `RestartCount` `1`; если PID 1 — оболочка (`sh -c "..."`), `kill -TERM 1` не действует, работает `kill -TERM -1` |

**Не проверено запуском:** выпуск сертификата у настоящего Let's Encrypt по домену во всех трёх вариантах (локально без публичного адреса не воспроизводится); `deploy.sh`, джоб `deploy` и `poll.sh` на сервере курса и на GitHub; срабатывание `cron` по расписанию на сервере. Сделанное на сервере курса — в [инструкции](practice-7-guide.md) этапы 2, 4 и 7.

---

## 1. `deploy/compose.yaml`: стек на сервере

Отличия от `compose.yaml` для ноутбука: образ берётся из реестра по тегу из `.env`, а не собирается (`build: .`); у `app` и `db` нет `ports:` — наружу открыт только прокси; у всех сервисов `restart: unless-stopped`; у `app` — ротация логов. Опубликованный порт Docker открывает мимо межсетевого экрана ufw ([Docker Docs: Packet filtering and firewalls](https://docs.docker.com/engine/network/packet-filtering-firewalls/), раздел **Docker and ufw**) — поэтому порт закрывается отсутствием `ports:`, а не правилом ufw.

`compose.yaml` в корне репозитория не трогайте: он остаётся для ноутбука.

```yaml
# Стек на сервере (практика №7, ADR 006). Значения — только в deploy/.env рядом с этим файлом.
# Запуск — в папке deploy на сервере: docker compose pull, затем docker compose up -d.
# Строки под команду помечены комментарием (таблица под блоком в шаблоне); пометку после правки удалите.

services:
  # Единственный сервис с портами наружу (ADR 006, п. 1–2). Caddy; Nginx и Traefik — шаблон, раздел 2.
  proxy:
    image: caddy:2.11.9
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    environment:
      DOMAIN: ${DOMAIN:?set DOMAIN in .env}
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile:ro
      - proxy_data:/data   # сертификат и ключ: без тома каждый запуск выпускает сертификат заново
    depends_on:
      - app

  # Портов наружу нет: прокси ходит к app по внутренней сети Compose, по имени сервиса.
  app:
    image: ${APP_IMAGE:?set APP_IMAGE in .env}:${APP_TAG:?set APP_TAG in .env}
    restart: unless-stopped
    environment:
      APP_ENV: prod
      LOG_LEVEL: ${LOG_LEVEL:-INFO}
      # АДАПТИРОВАТЬ: имя базы — как в compose.yaml; остальные переменные prod — ADR 003, приложение А
      DATABASE_URL: postgresql://postgres:${POSTGRES_PASSWORD:?set POSTGRES_PASSWORD in .env}@db:5432/app
    depends_on:
      db:
        condition: service_healthy
    logging:
      driver: local   # ротация: до 5 файлов по 10 МБ; строки живут, пока жив контейнер
      options:
        max-size: "10m"
        max-file: "5"

  db:
    image: postgres:16   # АДАПТИРОВАТЬ: версия — как в compose.yaml
    restart: unless-stopped
    environment:
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:?set POSTGRES_PASSWORD in .env}
      POSTGRES_DB: app   # АДАПТИРОВАТЬ: как в compose.yaml
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -h localhost -U postgres -d app"]   # АДАПТИРОВАТЬ: как в compose.yaml
      interval: 5s
      timeout: 3s
      retries: 10
      start_period: 30s

volumes:
  pgdata:
  proxy_data:
```

| Что | Где | Что сделать |
|---|---|---|
| Имя базы | `DATABASE_URL`, `POSTGRES_DB`, `healthcheck` у `db` | Как в вашем `compose.yaml` (у проектов курса — например, `repair_requests`) |
| Пользователь базы | `db` → `environment`, `DATABASE_URL`, `pg_isready -U` | Если в `compose.yaml` задан `POSTGRES_USER` — та же строка в `db` → `environment` и то же имя в `DATABASE_URL` (вместо `postgres`) и в `pg_isready -U`. Во всех трёх местах одно имя, иначе `role "..." does not exist` |
| Версия PostgreSQL | `db` → `image` | Как в `compose.yaml` |
| Другие обязательные переменные модуля настроек | `app` → `environment` | Все из колонки `prod` приложения А ADR 003 (приложения А нет — поля модуля настроек без умолчания). Секрет — строкой `ИМЯ: ${ИМЯ:?set ИМЯ in .env}` и значением в `deploy/.env`; не секрет — значением прямо здесь |
| `DATABASE_URL` | `app` → `environment` | Схема остаётся `postgresql://` (не `postgresql+psycopg://`): так её читают `/health` и `migrations/env.py` |
| Миграции при старте стека | — | Раздел 7 |
| Прокси Nginx или Traefik | `proxy` | Раздел 2.2 или 2.3 |
| Логи дольше жизни контейнера | `app` → `logging` | Домашнее задание 2 практики, раздел 9 [шаблона логирования](logging-template.md) |

Переменные `${...:?...}` Compose берёт из файла `.env` в папке первого compose-файла, то есть из `deploy/.env` ([Docker Docs: Interpolation](https://docs.docker.com/compose/how-tos/environment-variables/variable-interpolation/), раздел о файле `.env`, 3 мин). Переменной нет — Compose останавливается с ошибкой и ничего не запускает: тот же fail fast, что у модуля настроек.

---

## 2. Конфигурация reverse-proxy

Все три варианта: HTTPS с сертификатом Let's Encrypt для поддомена команды, `http://` перенаправляется на `https://`, запросы уходят в `app:8000`. Let's Encrypt проверяет, что домен ваш, запросом на порт 80 сервера (HTTP-01) — поэтому прокси запускается только после того, как поддомен указывает на сервер. Лимиты Let's Encrypt: 5 неудачных проверок одного имени в час, 5 сертификатов на одно имя за 7 дней ([Let's Encrypt: Rate Limits](https://letsencrypt.org/docs/rate-limits/), 5 мин). Отсюда два правила: том прокси (`proxy_data`) не удаляйте — в нём выданный сертификат; при ошибке выпуска не перезапускайте прокси раз за разом, а читайте его лог.

### 2.1. Caddy (путь по умолчанию)

Сервис `proxy` — как в разделе 1. Файл `deploy/Caddyfile`:

```text
# Caddy: HTTPS для поддомена команды; сертификат Let's Encrypt Caddy выпускает и продлевает сам.
# Имя сайта — переменная DOMAIN из deploy/.env (передаётся в контейнер proxy).
{$DOMAIN} {
	reverse_proxy app:8000
}
```

Caddy сам получает сертификат для имени сайта и перенаправляет `http://` на `https://` (`308`); заголовки `X-Forwarded-For` и `X-Forwarded-Proto` добавляет `reverse_proxy` ([Caddy: Automatic HTTPS](https://caddyserver.com/docs/automatic-https), разделы **Overview** и **ACME challenges**, 5 мин; `{$DOMAIN}` — [Caddyfile Concepts](https://caddyserver.com/docs/caddyfile/concepts), раздел **Environment variables**). Сертификат получен — в `docker compose logs proxy` строка с `certificate obtained successfully` (текст строки курсом не проверен: выпуск Let's Encrypt локально не воспроизводится).

**Let's Encrypt не выдаёт сертификат до конца занятия** (лимит неудачных проверок, DNS) — временно сертификат внутреннего удостоверяющего центра Caddy: строка `tls internal` внутри блока сайта, перед `reverse_proxy` — коммитом в рабочую ветку, не правкой на сервере (команды — инструкция, шаг 2.4); на сервере `git pull` и `docker compose up -d --force-recreate proxy`. Сертификат действует 12 часов и перевыпускается сам (прогон инструкции 2026-11-11, издатель `Caddy Local Authority - ECC Intermediate`). Браузер и `curl` такому сертификату не доверяют (`curl` — только с `-k`); `deploy.sh` — только со строкой `CURL_INSECURE=1` в `deploy/.env` (раздел 4). Это отклонение от п. 1 «Решения»: строка в «Рисках» ADR 006, возврат к Let's Encrypt — ДЗ 1 (строку `tls internal` удалить PR, `CURL_INSECURE=1` — из `deploy/.env`). Пункт DoD «Настроен HTTPS» временный сертификат не закрывает. Проверено для `localhost`, IP-адреса (раздел 8) и поддомена на имитации сервера (прогон инструкции).

### 2.2. Nginx и certbot

Nginx сертификат не выпускает: это делает certbot, один раз, до первого запуска `proxy` (ему нужен свободный порт 80). Сервис `proxy` в `deploy/compose.yaml` замените двумя сервисами:

```yaml
  # Nginx: сертификат выпускает certbot до первого запуска proxy (шаг 2.3), файлы — в томе proxy_data.
  proxy:
    image: nginx:1.30.6
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    environment:
      DOMAIN: ${DOMAIN:?set DOMAIN in .env}
    volumes:
      - ./nginx.conf.template:/etc/nginx/templates/default.conf.template:ro
      - proxy_data:/etc/letsencrypt:ro
    depends_on:
      - app

  # Выпуск сертификата: docker compose run --rm --service-ports certbot certonly ... (шаблон, раздел 2.2).
  certbot:
    image: certbot/certbot:v5.10.0
    profiles: ["certbot"]   # командой up не запускается
    ports:
      - "80:80"
    volumes:
      - proxy_data:/etc/letsencrypt
```

Файл `deploy/nginx.conf.template` — образ Nginx при старте подставляет в него переменные окружения и кладёт результат в конфигурацию ([nginx на Docker Hub](https://hub.docker.com/_/nginx), раздел **Using environment variables in nginx configuration**, 3 мин):

```text
# Nginx: HTTPS для поддомена команды. ${DOMAIN} подставляется при старте контейнера из переменной DOMAIN.
server {
    listen 80;
    server_name ${DOMAIN};
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name ${DOMAIN};

    ssl_certificate     /etc/letsencrypt/live/${DOMAIN}/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/${DOMAIN}/privkey.pem;

    # Адрес app — при каждом запросе через DNS Docker: после пересоздания контейнера у app может быть новый IP
    resolver 127.0.0.11 valid=10s;
    set $app_upstream http://app:8000;

    location / {
        proxy_pass $app_upstream;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

`resolver` и переменная в `proxy_pass` нужны из-за деплоя: с адресом `http://app:8000` прямо в `proxy_pass` Nginx запоминает IP контейнера при старте, и после пересоздания `app` с новым адресом отвечал бы `502`, пока его не перезапустят ([Nginx: proxy_pass](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass) — абзац о переменных и `resolver`).

Выпуск сертификата — на сервере, в папке `deploy`, **до** первого `docker compose up -d` (или после `docker compose stop proxy`); `<поддомен>` — значение `DOMAIN`:

```bash
docker compose run --rm --service-ports certbot certonly --standalone -d <поддомен> --agree-tos --register-unsafely-without-email --non-interactive
```

**Ожидаемый вывод:** в конце `Successfully received certificate.` и пути `/etc/letsencrypt/live/<поддомен>/fullchain.pem`. Этой строки нет (ошибка Let's Encrypt) — `proxy` не запускайте: Nginx без файлов сертификата перезапускается по кругу (`cannot load certificate`). Один повтор после исправления причины; не выдан — блок «Временный сертификат» ниже. Первый запуск скачивает образ certbot — около минуты вместе с выпуском (локально 60 с до ответа Let's Encrypt). Сертификат действует 90 дней — до конца курса продлевать не нужно; продление — строка в «Рисках» ADR 006 ([Certbot: Standalone](https://eff-certbot.readthedocs.io/en/stable/using.html#standalone)).

**Временный сертификат** — если Let's Encrypt не выдал сертификат к 45-й минуте (инструкция, шаг 2.4). Самоподписанный сертификат на 7 дней кладётся в том `proxy_data` по тому же пути, что у certbot; `<поддомен>` — значение `DOMAIN`:

```bash
docker compose run --rm --no-deps --entrypoint sh certbot -c 'mkdir -p "/etc/letsencrypt/live/$0" && openssl req -x509 -newkey rsa:2048 -nodes -days 7 -subj "/CN=$0" -addext "subjectAltName=DNS:$0" -keyout "/etc/letsencrypt/live/$0/privkey.pem" -out "/etc/letsencrypt/live/$0/fullchain.pem"' <поддомен>
docker compose up -d
```

**Ожидаемый вывод:** строка из точек и плюсов (генерация ключа), затем `-----`; `docker compose ps` — `proxy` `Up`, без перезапусков. `curl` с ноутбука — с `-k`; в `deploy/.env` — строка `CURL_INSECURE=1` (раздел 4). Перед выпуском Let's Encrypt (ДЗ 1) временный сертификат удалите. Иначе certbot остановится с ошибкой `live directory exists for <поддомен>` — уже после того, как Let's Encrypt выпустил сертификат (он засчитан в лимит 5 на тот же набор имён за 7 дней), — и оставит пустой файл `renewal/<поддомен>.conf`; если затем удалить только `live/<поддомен>`, certbot положит сертификат в `live/<поддомен>-0001`, а Nginx его не увидит. Команда ниже удаляет оба места и подходит для обоих случаев (до выпуска и после такой ошибки):

```bash
docker compose stop proxy
docker compose run --rm --no-deps --entrypoint rm certbot -rf /etc/letsencrypt/live/<поддомен> /etc/letsencrypt/renewal/<поддомен>.conf
```

Затем — выпуск командой выше и `docker compose up -d`. Курсом проверено (2026-11-11): создание и удаление — локально для `localhost`; выпуск certbot v5.10.0 после удаления, ошибка `live directory exists` без удаления и выпуск после неё этой командой — с тестовым ACME-сервером (Pebble), не с Let's Encrypt.

### 2.3. Traefik

Traefik берёт настройки маршрутов из меток (`labels`) сервисов, поэтому меняются два места `deploy/compose.yaml`. Сервис `proxy` замените блоком:

```yaml
  # Traefik находит app по меткам (labels) у сервиса app; сертификат — Let's Encrypt, проверка HTTP-01 на порту 80.
  proxy:
    image: traefik:v3.7.18
    restart: unless-stopped
    command:
      - --providers.docker=true
      - --providers.docker.exposedbydefault=false
      - --entrypoints.web.address=:80
      - --entrypoints.web.http.redirections.entrypoint.to=websecure
      - --entrypoints.web.http.redirections.entrypoint.scheme=https
      - --entrypoints.websecure.address=:443
      - --certificatesresolvers.le.acme.httpchallenge=true
      - --certificatesresolvers.le.acme.httpchallenge.entrypoint=web
      - --certificatesresolvers.le.acme.storage=/letsencrypt/acme.json
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro   # чтение списка контейнеров
      - proxy_data:/letsencrypt                          # acme.json: сертификат и ключ
    depends_on:
      - app
```

В сервис `app` перед строкой `logging:` добавьте метки (отступ — четыре пробела, как у `environment:`):

```yaml
    labels:   # для Traefik: какой адрес ведёт в этот сервис
      - traefik.enable=true
      - traefik.http.routers.app.rule=Host(`${DOMAIN:?set DOMAIN in .env}`)
      - traefik.http.routers.app.entrypoints=websecure
      - traefik.http.routers.app.tls.certresolver=le
      - traefik.http.services.app.loadbalancer.server.port=8000
```

Без `traefik.enable=true` Traefik сервис не видит: стоит `exposedbydefault=false` ([Traefik & Docker](https://doc.traefik.io/traefik/reference/install-configuration/providers/docker/), параметр `exposedByDefault`; [Traefik: ACME](https://doc.traefik.io/traefik/reference/install-configuration/tls/certificate-resolvers/acme/), раздел **httpChallenge**, 5 мин). В документации Traefik `acme.email` отмечен обязательным, но Traefik v3.7.18 и без него зарегистрировался в Let's Encrypt (проверка 2026-11-11). Если в `docker compose logs proxy` ошибка об email — добавьте строку `--certificatesresolvers.le.acme.email=<email владельца>`: email — не секрет, но персональные данные в публичном репозитории; согласуйте в команде. Пока сертификата нет, Traefik отдаёт свой самоподписанный `TRAEFIK DEFAULT CERT` — это и есть временный сертификат шага 2.4 инструкции: ничего не меняйте, `curl` — с `-k`, в `deploy/.env` — `CURL_INSECURE=1` (раздел 4).

Том `/var/run/docker.sock` даёт Traefik доступ к Docker сервера — строка в «Рисках» ADR 006.

---

## 3. `deploy/.env.example`

Пример файла переменных сервера — в git; сам `deploy/.env` — только на сервере.

```text
# Пример deploy/.env. На сервере: скопировать в deploy/.env и задать значения (практика №7, шаг 2.2).
# Файл .env в git не попадает.
# Поддомен команды из рассылки курса
DOMAIN=team.example.org
# Образ из Packages: ghcr.io/<владелец>/<репозиторий>, только строчные буквы
APP_IMAGE=ghcr.io/owner/repository
# Развёрнутый тег: sha- и 40 символов SHA коммита в main
APP_TAG=sha-0000000000000000000000000000000000000000
# Пароль базы: на сервере — случайная строка из латинских букв и цифр
POSTGRES_PASSWORD=changeme
# Уровень логов: DEBUG, INFO, WARNING или ERROR
LOG_LEVEL=INFO
# Только при временном сертификате (инструкция, шаг 2.4): deploy.sh проверяет /health без проверки сертификата
# CURL_INSECURE=1
```

Другие переменные `prod` из раздела 1 — сюда же строками с примером значения. gitleaks 8.28.0 и detect-secrets 1.5.0 по этому файлу — чисто (проверка 2026-11-11).

Проверить, что `deploy/.env` в git не попадёт: `git check-ignore -v deploy/.env` выводит строку `.gitignore` с правилом `.env`. Пусто — добавьте в `.gitignore` строку `.env` (без пути: правило действует в любой папке).

`.gitattributes` в корне репозитория — строка, чтобы скрипты на сервере не получили переводы строк Windows (иначе `sh` падает с `$'\r': command not found`):

```text
*.sh text eol=lf
```

---

## 4. `deploy/deploy.sh`: деплой по тегу

Один скрипт для всех способов п. 4: способ A — его запускает участник по SSH, B — джоб `deploy`, C — `poll.sh`. Это и «скрипт для автоматического деплоя» из DoD 5, и команда отката: прошлый тег — тот же вызов.

```sh
#!/bin/sh
# Деплой образа по тегу: sh deploy.sh sha-<40 символов SHA коммита в main>
# Записывает тег в deploy/.env, скачивает образ, пересоздаёт app, ждёт healthy
# и проверяет /health через прокси. Код выхода 0 — новая версия отвечает 200.
set -eu
cd "$(dirname "$0")"

TAG="${1:-}"
case "$TAG" in
  sha-????????????????????????????????????????) ;;
  *) echo "Использование: sh deploy.sh sha-<40 символов SHA>" >&2; exit 2 ;;
esac

PREV=$(sed -n 's/^APP_TAG=//p' .env)
DOMAIN=$(sed -n 's/^DOMAIN=//p' .env)

sed -i "s/^APP_TAG=.*/APP_TAG=$TAG/" .env
if ! docker compose pull app; then
  sed -i "s/^APP_TAG=.*/APP_TAG=$PREV/" .env
  echo "Образа с тегом $TAG нет в реестре: publish ещё не закончился или тег с ошибкой. Тег в .env возвращён: $PREV" >&2
  exit 1
fi
if ! docker compose up -d --wait --wait-timeout 120 app; then
  echo "app не стал healthy за 120 с. Откат: sh deploy.sh $PREV" >&2
  exit 1
fi

# Временный сертификат (инструкция, шаг 2.4): CURL_INSECURE=1 в .env — /health без проверки сертификата
INSECURE=$(sed -n 's/^CURL_INSECURE=//p' .env)
set -- -sS -o /dev/null -w '%{http_code}' --resolve "$DOMAIN:443:127.0.0.1"
[ "$INSECURE" = "1" ] && set -- "$@" -k
# Прокси начинает вести запросы в новый контейнер не сразу после healthy (Traefik — до 15 с):
# до 15 попыток с паузой 2 с; ошибку curl показывает только последняя
N=1
CODE=$(curl "$@" "https://$DOMAIN/health" 2>/dev/null || true)
while [ "$CODE" != "200" ] && [ "$N" -lt 15 ]; do
  sleep 2
  N=$((N + 1))
  CODE=$(curl "$@" "https://$DOMAIN/health" 2>/dev/null || true)
done
[ "$CODE" = "200" ] || CODE=$(curl "$@" "https://$DOMAIN/health" || true)
echo "Было: $PREV"
echo "Стало: $TAG, /health через прокси: $CODE"
[ "$INSECURE" = "1" ] && echo "Сертификат не проверялся: CURL_INSECURE=1 в .env (временный сертификат)"
[ "$CODE" = "200" ]
```

- `--wait` ждёт, пока `app` станет `healthy` по `HEALTHCHECK` образа ([docker compose up](https://docs.docker.com/reference/cli/docker/compose/up/), параметры `--wait` и `--wait-timeout`)
- Проверка `/health` повторяется до 15 раз через 2 с: Traefik включает маршрут к новому контейнеру через 0,5–14 с после `healthy` (прогон инструкции на имитации сервера), и один запрос сразу после `--wait` получал `404`. Ответ не `200` и через 30 с — проблема настоящая: скрипт печатает код последней попытки и ошибку `curl`, если она была
- `--resolve` отправляет запрос к прокси на этом же сервере по имени поддомена: проверяются сертификат, прокси и приложение, без зависимости от того, как сервер видит свой публичный адрес
- `CURL_INSECURE=1` в `deploy/.env` — только при временном сертификате (инструкция, шаг 2.4): проверка `/health` идёт с `-k`, и скрипт пишет об этом строкой `Сертификат не проверялся`. Без строки (путь по умолчанию) сертификат проверяется строго. Удалите её вместе с переходом на Let's Encrypt
- Запускается через `sh`, а не `./deploy.sh`: на Windows git не сохраняет у файла признак исполняемого
- `app` не стал `healthy` — тег в `.env` уже новый, старый контейнер удалён; скрипт печатает команду отката. Сообщение «за 120 с» бывает и раньше: `--wait` прекращает ожидание, как только `app` получил `unhealthy` (строка `container deploy-app-1 is unhealthy`). В ADR 006 (п. 5) это время и есть «за сколько вернуть прошлую версию». Повторный запуск с тем же новым тегом и снова неудача — в строке `Откат:` уже этот же тег (он записан в `.env`): откатывайтесь на тег из строки `Откат:` первого запуска или из строки `Было:` прошлого успешного деплоя

---

## 5. Способ B: джоб `deploy` в CI

Джоб идёт после `publish`, с тем же условием `if` — только `push` в `main`: подключается к серверу по SSH и вызывает `deploy.sh` с тегом этого коммита. Сторонних actions нет: у части команд **Actions permissions** разрешают только список издателей. Вставьте в конец `.github/workflows/ci.yml`, после `publish` и одной пустой строки; отступ — два пробела перед `# ----` и `deploy:`, как у других джобов. Шаблон CI курса (`github-actions-template.yml`) не меняется — джоб только в вашем `ci.yml`.

```yaml
  # ---------------------------------------------------------------------------
  # deploy — способ B ADR 006: после publish сервер разворачивает образ этого коммита
  # скриптом deploy/deploy.sh. Только push в main; обязательной проверкой не бывает.
  # Секреты DEPLOY_* задаёт владелец репозитория до практики №7.
  # ---------------------------------------------------------------------------
  deploy:
    needs: publish
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    timeout-minutes: 10
    concurrency: deploy                 # два деплоя подряд не идут одновременно
    steps:
      - name: SSH key and known host
        env:
          DEPLOY_SSH_KEY: ${{ secrets.DEPLOY_SSH_KEY }}
          DEPLOY_KNOWN_HOSTS: ${{ secrets.DEPLOY_KNOWN_HOSTS }}
        run: |
          install -m 700 -d ~/.ssh
          printf '%s\n' "$DEPLOY_SSH_KEY" > ~/.ssh/deploy_key
          chmod 600 ~/.ssh/deploy_key
          printf '%s\n' "$DEPLOY_KNOWN_HOSTS" > ~/.ssh/known_hosts

      - name: Deploy
        env:
          DEPLOY_HOST: ${{ secrets.DEPLOY_HOST }}
          DEPLOY_USER: ${{ secrets.DEPLOY_USER }}
          REPO_DIR: ${{ github.event.repository.name }}   # папка клона на сервере (шаг 2.1)
          TAG: sha-${{ github.sha }}
        run: >-
          ssh -i ~/.ssh/deploy_key "$DEPLOY_USER@$DEPLOY_HOST"
          "cd ~/$REPO_DIR && git fetch --prune origin && git checkout -B main origin/main && sh deploy/deploy.sh $TAG"
```

| Секрет | Значение | Откуда |
|---|---|---|
| `DEPLOY_HOST` | Поддомен команды | Рассылка курса |
| `DEPLOY_USER` | Имя пользователя на сервере | Рассылка курса |
| `DEPLOY_SSH_KEY` | Закрытый ключ деплоя целиком, со строками `-----BEGIN ...` и `-----END ...` | Инструкция, «Подготовка», блок «Только способ B» |
| `DEPLOY_KNOWN_HOSTS` | Строка `<поддомен> ssh-ed25519 AAAA...` | Там же |

Секреты создаёт владелец репозитория: **Settings → Secrets and variables → Actions → New repository secret** ([Using secrets in GitHub Actions](https://docs.github.com/en/actions/how-tos/write-workflows/choose-what-workflows-do/use-secrets), раздел **Creating secrets for a repository**, 3 мин). Значения секретов в логе прогона заменяются на `***`.

- `deploy` не добавляйте в обязательные проверки ruleset: на PR он пропущен по `if`, а пропущенный GitHub считает успешным
- На сервере джоб переключает клон на `main` (`git checkout -B main origin/main`): файлы `deploy/` на сервере всегда равны `main`, а `deploy/.env` не в git и не меняется. Файлы клона на сервере руками не правят — иначе `git checkout` остановится с `Your local changes ... would be overwritten`
- Ключ деплоя даёт выполнение любой команды на сервере под пользователем команды (а он в группе `docker`) — строка в «Рисках» ADR 006. Опция `restrict` в `authorized_keys` (блок «Только способ B» инструкции) снимает интерактивный терминал и пробросы, но не ограничивает команду

---

## 6. Способ C: `deploy/poll.sh` и расписание

Сервер раз в 2 минуты сверяет SHA последнего коммита `main` с тегом в `deploy/.env`; новый коммит и его образ уже в реестре — вызывает `deploy.sh`. Секретов в GitHub нет; публичный пакет скачивается без входа в реестр.

```sh
#!/bin/sh
# Способ C ADR 006: сервер сам забирает новую версию. Запускается по расписанию (cron).
# Новый коммит в main и его образ уже в реестре — deploy/deploy.sh с тегом этого коммита.
set -eu
cd "$(dirname "$0")/.."

git fetch --quiet origin main
TAG="sha-$(git rev-parse origin/main)"
[ "$TAG" = "$(sed -n 's/^APP_TAG=//p' deploy/.env)" ] && exit 0

IMAGE=$(sed -n 's/^APP_IMAGE=//p' deploy/.env)
docker pull --quiet "$IMAGE:$TAG" > /dev/null 2>&1 || exit 0   # publish ещё не закончился

git checkout --quiet -B main origin/main
echo "$(date -Is) $TAG"
sh deploy/deploy.sh "$TAG"
```

Расписание — на сервере, после merge (инструкция, шаг 7.1); `<репозиторий>` — папка клона:

```bash
( crontab -l 2>/dev/null; echo '*/2 * * * * flock -n $HOME/poll.lock sh $HOME/<репозиторий>/deploy/poll.sh >> $HOME/poll.log 2>&1' ) | crontab -
crontab -l
```

**Ожидаемый вывод** второй команды — строка `*/2 * * * * flock -n $HOME/poll.lock sh $HOME/...`. `flock -n` не запускает `poll.sh`, пока не закончился прошлый запуск: `deploy.sh` может идти дольше 2 минут (`pull` и ожидание `healthy`), и без него два деплоя одного тега шли бы одновременно. Журнал запусков — `tail ~/poll.log`: пусто, пока новых коммитов нет; после деплоя — время, тег и вывод `deploy.sh`. Кода выхода в журнале нет: успех — `200` в строке `Стало:`.

- **Откат при способе C:** сначала выключить расписание — `crontab -l | grep -v poll.sh | crontab -`, затем `sh deploy/deploy.sh <прошлый тег>`; иначе через 2 минуты `poll.sh` вернёт новый образ. Включить — команда выше. Это строка п. 5 ADR 006
- `deploy.sh` упал на `healthy` — тег в `.env` уже новый, `poll.sh` его не повторяет; упал на скачивании — тег возвращён, `poll.sh` попробует через 2 минуты

---

## 7. Миграции при старте стека на сервере

Как в вашем `compose.yaml` (домашнее задание 2 практики №6):

| Как сделано в `compose.yaml` | Что сделать в `deploy/compose.yaml` |
|---|---|
| Сервис `migrate`, `app` ждёт его через `depends_on` с `condition: service_completed_successfully` | Скопировать сервис `migrate` целиком; `build: .` заменить строкой `image:` из `app`; `environment` — как у `app`; `restart:` у `migrate` не добавлять (он завершается). У `app` в `depends_on` — то же условие, что в `compose.yaml`. `deploy.sh` с такой связкой работает: `up -d --wait app` запускает `migrate`, дожидается его завершения и `healthy` у `app` (проверено, Compose 5.5.1). При смене тега Compose пересоздаёт и `migrate` (тот же образ), а новый `app` стартует только после него: окно простоя шага 7.1 включает прогон миграций — 33–34 с в прогоне инструкции против нескольких секунд без `migrate` у Caddy и Nginx (у Traefik без `migrate` — 20–30 с: он ведёт запросы в `app` только после `healthy`) |
| Команда контейнера `app` (`command:` с `alembic upgrade head && uvicorn ...`) | Скопировать `command:` в `app`; в строку `uvicorn` добавить `--no-access-log`, как в `Dockerfile` ([шаблон логирования](logging-template.md), раздел 6) |
| Миграций при старте нет | Один раз после первого `up -d`: `docker compose run --rm app alembic upgrade head` (инструкция, шаг 2.3); миграции при старте — ДЗ |

---

## 8. Справка: сервер без домена

На практике №7 у каждой команды поддомен курса; этот раздел — для сервера только с IP-адресом (например, после курса).

- **Сертификат внутреннего удостоверяющего центра Caddy для IP.** Проверено на сервере курса в частной сети (Caddy v2.11.9, Docker 29.9.2, 2026-11-11): браузер и `curl` без `-k` сертификату не доверяют; без глобального блока `default_sni` клиенты, обращающиеся по IP, получают обрыв TLS (`curl: (35)`): они не передают имя сервера, и Caddy ищет сертификат для адреса контейнера в сети Docker

```text
{
	default_sni 203.0.113.10
}

203.0.113.10 {
	tls internal
	reverse_proxy app:8000
}
```

`203.0.113.10` — пример, подставьте IP сервера в обе строки ([Caddy: Global options](https://caddyserver.com/docs/caddyfile/options), `default_sni`).

- **Let's Encrypt для IP-адреса** — с 15.01.2026 только короткие сертификаты на 160 часов, профиль `shortlived` ([Let's Encrypt: 6-day and IP Address Certificates are Generally Available](https://letsencrypt.org/2026/01/15/6day-and-ip-general-availability/)). Caddy автоматически для IP публичный сертификат не запрашивает. Курсом не проверено

---

## История изменений

| Дата | Что изменилось | Что это значит для команд |
|---|---|---|
| 2026-11-11 | Первая версия | — |
