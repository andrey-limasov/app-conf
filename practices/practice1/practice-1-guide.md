# Практика №1: "Инфраструктура команды и первый коммит"

**Длительность:** 4 академических часа (180 минут)  
**Формат:** Самостоятельная командная работа по письменной инструкции  
**Роль преподавателя:** Организационная поддержка: точки синхронизации, приём приглашений в репозитории, фиксация прогресса

---

## Цели практики

К концу занятия каждая команда должна:

1. **Выбрать** тему проекта из идей домашнего задания №1 и зафиксировать её в `README.md`: название, основная сущность, пользователи, планируемые эндпоинты (шаги 1.1, 2.3, задача Б этапа 3)
2. **Создать** публичный репозиторий команды, в котором все участники и преподаватель `andrey-limasov` — коллабораторы, а приглашения сокомандников приняты (шаг 1.2)
3. **Настроить** Git на ноутбуке каждого участника так, чтобы коммиты подписывались его именем и email из GitHub, а `git push` проходил без пароля от аккаунта (подготовка, проверка — шаг 1.3)
4. **Оформить** каркас репозитория (`.gitignore`, `README.md`, `CONTRIBUTING.md`) первым коммитом по Conventional Commits, в который не попала папка `venv/` (этап 2)
5. **Создать** FastAPI-приложение, отвечающее `200` на `GET /health`, и зафиксировать зависимости файлами выбранного менеджера так, чтобы сокомандник установил их на своём ноутбуке и запустил приложение (задача К этапа 3, шаг 3.4)
6. **Провести** изменения через параллельные feature-ветки: у каждого участника свой Pull Request с approval от другого участника, конфликт двух PR в одном файле разрешён без потери изменений (этап 3)

---

## Подготовка

### Что должно быть готово до начала занятия

| Что | Откуда | Как проверить |
|---|---|---|
| Команда из 3–4 человек | Воркшоп №1, [домашнее задание №1](../../home-work-1.md), задание 1 | Все участники знают состав команды |
| Три идеи проекта | [Домашнее задание №1](../../home-work-1.md), задание 2 | Идеи записаны в чате команды или в файле |
| Аккаунт GitHub у **каждого** участника | [github.com/signup](https://github.com/signup) | Каждый знает свой логин и может войти |
| Git настроен, вход в GitHub из терминала работает | Раздел «Настройка Git и вход в GitHub» ниже | Проверки этого раздела проходят |
| ПО на ноутбуках (ниже) | Установка дома | Команды проверки ниже выводят версии |

В курсе используется **GitHub**: инструкции практик написаны для него, практика №2 и следующие опираются на его настройки (правила защиты веток, Pull Request, API). [Definition of Done](../../definition-of-done.md) допускает и GitLab, но шаги для него в практиках не даются — команда на GitLab переводит их сама.

### Обязательное ПО

- **Python 3.11 или новее** — [python.org/downloads](https://www.python.org/downloads/). В Windows python.org предлагает Python install manager: после его установки работают команды `python` и `py`, а сам Python ставится командой `py install 3.14`. Предложение добавить папку в PATH принимать необязательно. Если у вас классический установщик — отметьте в нём «Add python.exe to PATH»
- **Git** — [git-scm.com/downloads](https://git-scm.com/downloads). В Windows вместе с Git ставится Git Credential Manager — он нужен для входа в GitHub из терминала
- **macOS и Linux: GitHub CLI** (`gh`) — [cli.github.com](https://cli.github.com/), для входа в GitHub из терминала (см. «Настройка Git и вход в GitHub» ниже)
- **VS Code** или другой редактор — [code.visualstudio.com](https://code.visualstudio.com/)

### Проверка установки

Терминал: в Windows — **PowerShell**, в macOS и Linux — **Terminal**.

**Windows PowerShell:**

```powershell
python --version
git --version
```

**macOS / Linux:**

```bash
python3 --version
git --version
gh --version
```

**Ожидаемый вывод:** `Python 3.11.x` или новее (3.12, 3.13, 3.14 — тоже подходят), `git version 2.x`, `gh version 2.x`. Если GitHub CLI вы не ставите и будете входить по токену, `gh: command not found` — не ошибка. Запомните свою версию Python — она понадобится в `README.md`.

**Как проверить, что сработало:** все команды вывели версию, а не `command not found` / `is not recognized`. Если в Windows `python` открывает Microsoft Store — см. раздел «Если не работает».

### Настройка Git и вход в GitHub — каждый участник, до занятия

На занятии на это нет времени: шаг 1.3 только проверяет результат. Если не получилось — приходите с текстом ошибки, разберёте на шаге 1.3.

**1. Имя и email для коммитов.** Команды одинаковы во всех ОС:

```bash
git config --global user.name "Имя Фамилия"
git config --global user.email "email-из-вашего-аккаунта-GitHub"
```

Email должен совпадать с одним из адресов в GitHub (Settings → Emails), иначе коммиты не привяжутся к вашему аккаунту. Если там включено «Keep my email addresses private» — укажите адрес вида `12345678+login@users.noreply.github.com`, который GitHub показывает на той же странице.

**Как проверить:**

```bash
git config --global user.name
git config --global user.email
```

**Ожидаемый вывод:** ваше имя и ваш email — по строке на команду. Пустая строка означает, что значение не задано: повторите команды пункта 1.

**2. Вход в GitHub из терминала.** Пароль от аккаунта GitHub при `git push` не принимает.

**Windows.** Git Credential Manager входит в Git for Windows. При первом `git push` откроется окно входа в GitHub — выберите вход через браузер и подтвердите доступ. Сейчас проверьте, что он установлен:

```powershell
git credential-manager --version
```

**Ожидаемый вывод:** номер версии, например `2.9.0`.

**macOS / Linux:**

```bash
gh auth login
```

Ответы на вопросы: `GitHub.com` → `HTTPS` → `Authenticate Git with your GitHub credentials?` — `Yes` → `Login with a web browser`. Скопируйте одноразовый код из терминала, вставьте его в открывшейся странице и подтвердите доступ.

**Как проверить:**

```bash
gh auth status
```

**Ожидаемый вывод:** строка `Logged in to github.com account <ваш-логин>`. Если на вопрос про Git вы ответили `No` — выполните `gh auth setup-git`.

**Без GitHub CLI:** создайте personal access token (classic) с областью `repo` и вводите его вместо пароля при первом `git push` ([Managing your personal access tokens](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens)). Токен — тот же пароль: не публикуйте его в чатах и файлах репозитория.

GitHub CLI на Linux: команды установки для Debian / Ubuntu, Fedora и других дистрибутивов — [Installing gh on Linux](https://github.com/cli/cli/blob/trunk/docs/install_linux.md). Если ставить его некогда — войдите по токену. Чтобы не вводить токен при каждом push, на Linux без Git Credential Manager включите временное хранение в памяти: `git config --global credential.helper "cache --timeout=28800"` — при первом `git push` введите логин GitHub и токен вместо пароля, следующие 8 часов Git их не спросит.

**Что должно получиться:** у вас `git config` выводит его имя и email; в Windows установлен Git Credential Manager, в macOS / Linux `gh auth status` показывает вход.

**Полезные ссылки:**

- [Setting your commit email address (GitHub Docs)](https://docs.github.com/en/account-and-profile/setting-up-and-managing-your-personal-account-on-github/managing-email-preferences/setting-your-commit-email-address)
- [gh auth login (GitHub CLI manual)](https://cli.github.com/manual/gh_auth_login)

### Если ноутбук на команду один

Работать можно, но следы вклада каждого участника нужно оставить явно — по ним идёт оценка сокомандников ([assessment.md](../../assessment.md), раздел 3):

- **Коммит** — с указанием автора: `git commit --author="Имя Фамилия <email-из-GitHub>" -m "..."`. Push делается под тем входом, который настроен на ноутбуке: все участники — коллабораторы, поэтому push пройдёт
- **Pull Request** открывает сам участник из **своего** аккаунта: в окне инкогнито браузера или с телефона — Pull requests → New pull request → выбрать свою ветку
- **Approve** ставит ревьюер тоже из своего аккаунта. Автор PR поставить approval себе не может — это правило GitHub

### Если на занятии двое

Задачи этапа 3 распределяются так: участник 1 — задачи К и Б (две ветки, два PR), участник 2 — задача А. Ревьюят друг друга. Конфликт между А и Б возникнет так же, как в полной команде.

---

## Пошаговая инструкция

Общий тайминг: **180 минут**.

| Этап | Название | Минут |
|---|---|---|
| 1 | Тема, репозиторий, доступы | 35 |
| 2 | Каркас проекта и первый коммит | 40 |
| 3 | Параллельные ветки: у каждого свой PR | 80 |
| 4 | Самопроверка и фиксация | 15 |
| — | Буфер на вход в GitHub у тех, кто не настроил его дома, и разброс темпа | 10 |
| | **Итого** | **180** |

### Точки синхронизации

Преподаватель проверяет их вслух. Команда, отставшая больше чем на 10 минут, сообщает об этом сразу.

| Минута | Что должно быть сделано у всех команд |
|---|---|
| **35** | Все участники — коллабораторы с принятыми приглашениями; у каждого настроен Git и есть клон репозитория |
| **75** | Первый коммит с `.gitignore`, `README.md`, `CONTRIBUTING.md` виден на странице репозитория |
| **120** | У каждого участника открыт свой PR с назначенным ревьюером |
| **155** | Все PR смержены, конфликт разрешён, локальный `main` у всех совпадает с GitHub |

---

### Этап 1: Тема, репозиторий, доступы (35 мин)

#### Шаг 1.1: Выбрать тему и распределить роли (10 мин)

**Что сделать.**

1. Откройте три идеи из домашнего задания №1 и выберите **одну**. Критерии:
   - есть основная сущность, над которой строится CRUD (см. MVP в [техническом задании](../../project-requirements.md))
   - её можно сделать за семестр командой из 3–4 человек
   - каждый участник может одной фразой сказать, кто пользователь приложения
2. Если идей нет — возьмите одну из пяти тем раздела «Примеры тем проектов» [технического задания](../../project-requirements.md). На выбор — не больше 5 минут.
3. Запишите в чат команды:
   - название проекта
   - короткое имя латиницей для репозитория: строчные буквы и дефисы, например `task-tracker`
   - основную сущность, пользователей и 3–5 эндпоинтов из идеи
4. Распределите роли по таблице. Задачу К (код) возьмёт тот, у кого Python уже работает без проблем. По умолчанию владелец репозитория — тимлид, автор задачи К — Tech Lead: так они записываются в README и в раздел «Контакты» `CONTRIBUTING.md`; команда может договориться иначе.
5. Не ждите преподавателя — переходите к шагу 1.2. На этапе 1 он обойдёт команды и задаст вопросы о теме. Если тему придётся изменить, владелец переименует репозиторий: Settings → General → Repository name (GitHub перенаправляет старые ссылки на новое имя), а участники обновят адрес у себя: `git remote set-url origin <новый-адрес>`.

| Участник | Этап 1 | Этап 2 | Этап 3: задача и ветка | Этап 3: ревьюит |
|---|---|---|---|---|
| 1 — владелец репозитория | Создаёт репозиторий, приглашает коллабораторов (1.2) | Пишет файлы каркаса, делает первый коммит (2.2–2.5) | Б: раздел «Предметная область» в `README.md`, `feature/readme-domain` | PR участника 2 (К) |
| 2 | Проверяет настройку Git (1.3), принимает приглашение | Виртуальное окружение (2.1), готовит тексты разделов «Встречи команды» и «Контакты» | К: приложение и зависимости, `feature/initial-setup` | PR участника 3 (А) |
| 3 | Проверяет настройку Git (1.3), принимает приглашение | Виртуальное окружение (2.1), собирает строки раздела «Команда» | А: раздел «API» в `README.md`, `feature/readme-api` | В команде из 3 — PR участника 1 (Б), из 4 — PR участника 4 (В) |
| 4 (если есть) | Проверяет настройку Git (1.3), принимает приглашение | Виртуальное окружение (2.1), вычитывает каркас перед коммитом | В: чек-лист ревьюера в `CONTRIBUTING.md`, `feature/review-checklist` | PR участника 1 (Б) |

**Что должно получиться:** в чате команды записаны тема (название, сущность, пользователи, эндпоинты), короткое имя для репозитория и таблица ролей с именами.

#### Шаг 1.2: Создать репозиторий и пригласить коллабораторов (15 мин)

**Что сделать — владелец репозитория:**

1. На [github.com](https://github.com) нажать **New** (или «+» → **New repository**)
2. Заполнить:
   - **Repository name:** `app-conf-project-` + короткое имя из шага 1.1, например `app-conf-project-task-tracker`
   - **Description:** одно предложение о проекте
   - **Public**
   - README, `.gitignore` и лицензию **не добавлять**: репозиторий должен быть пустым, файлы вы создадите на этапе 2
3. Нажать **Create repository**
4. Settings → в боковом меню раздел **Access** → **Collaborators** → **Add people**
5. Ввести логин сокомандника, выбрать его в списке, нажать **Add `<логин>` to `<репозиторий>`**. GitHub может попросить подтвердить вход паролем или кодом 2FA
6. Повторить пункт 5 для **каждого** сокомандника и для преподавателя: **`andrey-limasov`**

В личном репозитории роль не выбирается: каждый коллаборатор получает право записи, а Settings доступен только владельцу.

**Что сделать — остальные участники:** принять приглашение по ссылке из письма GitHub или на странице `https://github.com/<логин-владельца>/<имя-репозитория>/invitations` → **Accept invitation**. Приглашение действует 7 дней.

**Зачем всем, а не только преподавателю:** без права записи участник не сможет запушить свою ветку, его нельзя выбрать ревьюером, а его approval не будет учитываться правилами защиты ветки, которые вы включите на практике №2.

**Что должно получиться:** у владельца в Settings → Collaborators — все сокомандники без пометки «Pending invite» и `andrey-limasov`. Приглашение преподавателя может оставаться в статусе «Pending invite» до конца дня — его принимает сам преподаватель.

**Как проверить, что сработало:** у каждого сокомандника в списке нет «Pending invite». Без отправленного приглашения преподавателю работа не засчитывается.

**Полезные ссылки:**

- [Creating a new repository (GitHub Docs)](https://docs.github.com/en/repositories/creating-and-managing-repositories/creating-a-new-repository)
- [Inviting collaborators to a personal repository (GitHub Docs)](https://docs.github.com/en/account-and-profile/setting-up-and-managing-your-personal-account-on-github/managing-access-to-your-personal-repositories/inviting-collaborators-to-a-personal-repository)

#### Шаг 1.3: Проверить настройку Git и вход в GitHub — каждый участник (5 мин)

Сокомандники выполняют этот шаг сразу, не дожидаясь, пока владелец закончит шаг 1.2.

**Что сделать.** Выполните проверки из раздела «Настройка Git и вход в GitHub» в «Подготовке»:

```bash
git config --global user.name
git config --global user.email
```

Windows — `git credential-manager --version`; macOS / Linux — `gh auth status` (если входите по токену, этой проверки нет: токен понадобится при первом `git push`).

**Ожидаемый вывод:** ваше имя и email из GitHub (пустая строка — не настроено); номер версии Git Credential Manager или строка `Logged in to github.com account <ваш-логин>`.

**Если проверка не прошла** — выполните настройку из «Подготовки» сейчас. Владелец делает это после шага 1.2; время берётся из буфера.

#### Шаг 1.4: Клонировать репозиторий — каждый участник (5 мин)

**Что сделать.**

1. На странице **вашего нового** репозитория (не репозитория курса) в блоке «Quick setup» скопируйте HTTPS-адрес вида `https://github.com/<логин-владельца>/app-conf-project-<тема>.git`
2. Откройте терминал в папке, где будете хранить проекты, и выполните (подставьте свой адрес и имя папки):

```bash
git clone https://github.com/<логин-владельца>/app-conf-project-<тема>.git
cd app-conf-project-<тема>
```

**Ожидаемый вывод:**

```text
Cloning into 'app-conf-project-<тема>'...
warning: You appear to have cloned an empty repository.
```

Предупреждение про пустой репозиторий — норма: файлов в нём ещё нет.

**Как проверить, что сработало:**

```bash
git remote -v
```

Две строки `origin` с адресом **вашего** репозитория — `(fetch)` и `(push)`.

**Полезные ссылки:**

- [Cloning a repository (GitHub Docs)](https://docs.github.com/en/repositories/creating-and-managing-repositories/cloning-a-repository)

**Точка синхронизации 35:** все приглашения приняты, у каждого клон и настроенный Git.

---

### Этап 2: Каркас проекта и первый коммит (40 мин)

Шаг 2.1 выполняет каждый участник на своём ноутбуке. Файлы шагов 2.2–2.4 создаёт и коммитит владелец репозитория; остальные в это время готовят для него тексты по таблице ролей и вычитывают результат. Откройте папку репозитория в редакторе (VS Code: File → Open Folder).

#### Шаг 2.1: Виртуальное окружение — каждый участник (10 мин)

**Windows PowerShell:**

```powershell
python -m venv venv
.\venv\Scripts\Activate.ps1
```

Если активация завершилась ошибкой `running scripts is disabled on this system` (в русской Windows — `выполнение сценариев отключено в этой системе`) — один раз разрешите запуск локальных скриптов и повторите активацию:

```powershell
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
.\venv\Scripts\Activate.ps1
```

На вопрос о смене политики ответьте `Y`. В командной строке cmd вместо PowerShell активация — `venv\Scripts\activate.bat`.

**macOS / Linux:**

```bash
python3 -m venv venv
source venv/bin/activate
```

**Git Bash в Windows** (в том числе терминал VS Code с Git Bash): окружение создаётся командой `python -m venv venv`, активируется `source venv/Scripts/activate`.

**Ожидаемый вывод:** строка приглашения терминала начинается с `(venv)`.

**Как проверить, что сработало** (внутри окружения команда `python` есть во всех ОС):

```bash
python -c "import sys; print(sys.prefix)"
```

Путь заканчивается на `venv`. Окружение активируется заново в **каждом** новом окне терминала — той же командой активации.

**Полезные ссылки:**

- [Virtual Environments and Packages (Python Docs)](https://docs.python.org/3/tutorial/venv.html)

#### Шаг 2.2: Создать `.gitignore` (5 мин)

**Что сделать.** Создать в корне репозитория файл `.gitignore` с содержимым:

```gitignore
# Python
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
venv/
ENV/
env/
build/
develop-eggs/
dist/
downloads/
eggs/
.eggs/
lib/
lib64/
parts/
sdist/
var/
wheels/
*.egg-info/
.installed.cfg
*.egg

# Environment variables
.env
.env.local
.env.*.local

# IDE
.vscode/
.idea/
*.swp
*.swo
*~

# Database
*.db
*.sqlite
*.sqlite3

# Logs
*.log
logs/

# OS
.DS_Store
Thumbs.db

# Testing
.pytest_cache/
.coverage
htmlcov/
```

**Как проверить, что сработало:**

```bash
git status
```

**Ожидаемый вывод:** в `Untracked files` есть `.gitignore`, а `venv/` — нет. Если `venv/` в списке — проверьте, что файл называется ровно `.gitignore` (Windows может дописать `.txt`) и лежит в корне репозитория.

**Полезные ссылки:**

- [Ignoring files (GitHub Docs)](https://docs.github.com/en/get-started/getting-started-with-git/ignoring-files)

#### Шаг 2.3: Создать `README.md` (10 мин)

**Что сделать.** Создать `README.md` по шаблону. Каждую строку, где стоит `ЗАПОЛНИТЬ:`, замените **целиком** — вместе со словом `ЗАПОЛНИТЬ:` и подсказкой после него.

```markdown
# ЗАПОЛНИТЬ: название проекта

ЗАПОЛНИТЬ: 2–3 предложения — что делает приложение и для кого (из идеи, выбранной на шаге 1.1).

## Команда

- ЗАПОЛНИТЬ: Имя Фамилия (@github-логин) — роль в команде: тимлид, Tech Lead или разработчик
- ЗАПОЛНИТЬ: Имя Фамилия (@github-логин) — роль
- ЗАПОЛНИТЬ: Имя Фамилия (@github-логин) — роль

## Стек технологий

- Python ЗАПОЛНИТЬ: минимальная версия в команде по выводу `python --version` у всех участников, только две первые цифры, например 3.12
- FastAPI
- PostgreSQL
- Docker
- GitHub Actions

## Статус

Проект в разработке.

## Установка и запуск

(Будет добавлено позже)
```

Строк в разделе «Команда» — по числу участников. Раздел «Установка и запуск» оставьте как есть: его заполните на практике №3.

**Как проверить, что сработало:** в VS Code откройте предпросмотр (`Ctrl+Shift+V`, в macOS `Cmd+Shift+V`): заголовки отображаются заголовками, а не строками с `#`. Слова `ЗАПОЛНИТЬ` в файле не осталось — проверка командой будет на шаге 2.5.

**Полезные ссылки:**

- [Awesome README](https://github.com/matiassingers/awesome-readme) — примеры хороших README

#### Шаг 2.4: Создать `CONTRIBUTING.md` (10 мин)

**Что сделать.** Создать `CONTRIBUTING.md` по шаблону; строки с `ЗАПОЛНИТЬ:` обсудить командой и заменить целиком.

**Стратегию ветвления сейчас не выбирайте.** Её выбирают на воркшопе №2 после того, как команды разберут GitFlow, GitHub Flow и Trunk-Based, и фиксируют в ADR 001. До этого в файле стоит заглушка — оставьте её дословно. Названия разделов не меняйте: на них опирается практика №2.

```markdown
# Правила работы в команде

## Git-стратегия

Стратегия ветвления выбирается на воркшопе №2 и фиксируется в ADR 001.
До этого: ветки `feature/<кратко>` создаются от `main`, изменения попадают в `main` только через Pull Request.

### Ветвление
- `main` — стабильная версия
- `feature/<кратко>` — новые функции и документация
- `bugfix/<кратко>` — исправление ошибок

### Коммиты
Используем [Conventional Commits](https://www.conventionalcommits.org/):
- `feat:` новая функция
- `fix:` исправление бага
- `docs:` изменения в документации
- `style:` форматирование кода
- `refactor:` рефакторинг
- `test:` добавление тестов
- `chore:` рутинные задачи

Пример: `feat: add user authentication endpoint`

## Code Review

- Каждый PR требует минимум 1 approval от другого участника команды
- Ревьюер проверяет: стиль кода, архитектуру, наличие тестов
- Автор отвечает на все комментарии в PR

## Встречи команды

- Частота: ЗАПОЛНИТЬ: например, 2 раза в неделю
- Канал связи: ЗАПОЛНИТЬ: Telegram, Discord или другой, со ссылкой на чат
- Время ответа на сообщения: в течение 24 часов

## Разрешение конфликтов

1. Обсуждение в команде
2. Голосование (большинство голосов)
3. Если не можем решить — обращаемся к преподавателю

## Контакты

- Тимлид: ЗАПОЛНИТЬ: имя, Telegram — по умолчанию владелец репозитория (участник 1)
- Tech Lead: ЗАПОЛНИТЬ: имя, Telegram — по умолчанию автор задачи К (участник 2)
```

`<кратко>` в именах веток — не плейсхолдер, а часть правила: так записывается шаблон имени ветки.

**Полезные ссылки:**

- [Conventional Commits](https://www.conventionalcommits.org/) — стандарт написания коммитов

#### Шаг 2.5: Первый коммит и push (5 мин)

**Что сделать — владелец репозитория.**

1. Убедитесь, что плейсхолдеров не осталось.

   macOS / Linux:

   ```bash
   grep -n "ЗАПОЛНИТЬ" README.md CONTRIBUTING.md
   ```

   Windows PowerShell:

   ```powershell
   Select-String -Encoding utf8 -Pattern "ЗАПОЛНИТЬ" README.md, CONTRIBUTING.md
   ```

   **Ожидаемый вывод:** пусто. Найденные строки замените и повторите проверку.

2. Закоммитьте и отправьте:

```bash
git add .gitignore README.md CONTRIBUTING.md
git commit -m "docs: initial project structure and contributing guidelines"
git branch --show-current
git push -u origin main
```

`git branch --show-current` должна вывести `main`. Если вывела `master` — выполните `git branch -M main` и только потом `git push -u origin main`.

**Ожидаемый вывод `git push`:** в Windows при первом push откроется окно входа в GitHub («Настройка Git и вход в GitHub» в «Подготовке»). Затем:

```text
To https://github.com/<логин-владельца>/app-conf-project-<тема>.git
 * [new branch]      main -> main
branch 'main' set up to track 'origin/main'.
```

**Как проверить, что сработало:**

- на странице репозитория видны три файла и сообщение коммита
- `git log --format="%h %an %s"` выводит одну строку с вашим именем, а не с общим именем команды

Это **единственный прямой push в `main`** за курс: дальше все изменения идут через Pull Request, а на практике №2 прямой push в `main` запретит сам GitHub.

**Точка синхронизации 75.**

---

### Этап 3: Параллельные ветки: у каждого свой PR (80 мин)

Все участники одновременно создают свои ветки от первого коммита, работают параллельно, открывают PR и ревьюят PR друг друга. Задачи А и Б **обе дописывают раздел в конец `README.md`** — поэтому PR, который будет смержен вторым, получит конфликт. Это задумано: на шаге 3.5 вы его разрешите.

Порядок merge: сначала PR задачи К (код), затем PR задач А и Б — какой будет готов раньше. PR задачи В можно мержить в любой момент.

#### Шаг 3.1: Обновить `main` и создать свою ветку — каждый участник (5 мин)

**Что сделать.** Активируйте окружение (шаг 2.1) и выполните, подставив имя ветки своей задачи:

```bash
git pull origin main
git switch -c feature/<ваша-задача>
```

| Задача | Имя ветки |
|---|---|
| К — приложение и зависимости | `feature/initial-setup` |
| А — раздел «API» в `README.md` | `feature/readme-api` |
| Б — раздел «Предметная область» в `README.md` | `feature/readme-domain` |
| В — чек-лист ревьюера в `CONTRIBUTING.md` | `feature/review-checklist` |

**Если ваша команда уже склоняется к GitFlow** — сегодня всё равно создавайте ветки от `main`: ветки `develop` ещё нет, её заведёте после выбора стратегии на воркшопе №2.

**Как проверить, что сработало:** `git log --oneline` показывает первый коммит, `git branch --show-current` — имя вашей ветки.

**Полезные ссылки:**

- [Git Branching (Pro Git)](https://git-scm.com/book/en/v2/Git-Branching-Branches-in-a-Nutshell)

#### Шаг 3.2: Выполнить свою задачу (25 мин)

Каждый выполняет **только свою** задачу и не трогает другие файлы и разделы. Закончили раньше — помогите автору задачи К или заранее прочитайте PR, который будете ревьюить.

##### Задача К: приложение и зависимости

**К1. Установить зависимости.** Выберите один менеджер — окончательно его выберут на воркшопе №3 и зафиксируют в ADR 002.

**Вариант pip.**

macOS / Linux:

```bash
pip install fastapi "uvicorn[standard]"
pip freeze > requirements.txt
```

Windows PowerShell:

```powershell
pip install fastapi "uvicorn[standard]"
pip freeze | Out-File -Encoding utf8 requirements.txt
```

Кавычки вокруг `uvicorn[standard]` обязательны: в zsh (оболочка macOS по умолчанию) без них команда падает с `no matches found`. В Windows PowerShell 5.1 перенаправление `>` записывает файл в кодировке UTF-16 — GitHub покажет его в PR как двоичный, и ревьюер не увидит изменений. Поэтому в PowerShell — `Out-File -Encoding utf8`.

**Ожидаемый вывод:** строка `Successfully installed ...`, в ней есть `fastapi-` и `uvicorn-`. В `requirements.txt` — около 20 строк вида `пакет==версия`, среди них `fastapi==` и `uvicorn==`.

**Как проверить:** откройте `requirements.txt` в VS Code — текст читается, в строке состояния кодировка `UTF-8` или `UTF-8 with BOM`, а не `UTF-16 LE`.

**Вариант Poetry или uv (рекомендуется).** Ставится в то же окружение `venv/`; окружение должно быть активно.

Poetry:

```bash
pip install poetry
poetry init --no-interaction --python ">=3.11"
poetry add fastapi "uvicorn[standard]"
```

`--python ">=3.11"` обязателен: без него Poetry запишет в `pyproject.toml` версию вашего Python, и сокомандник с более старой версией не сможет установить зависимости. `--no-interaction` пропускает диалог из десятка вопросов. В `pyproject.toml` попадут имя папки как `name` и ваши имя и email из `git config` как `authors` — так и должно быть.

**Ожидаемый вывод:** десятки строк `Installing ...` (это не ошибка), последняя строка — `Writing lock file`; появились файлы `pyproject.toml` и `poetry.lock`.

uv:

```bash
pip install uv
uv init --bare --python ">=3.11"
uv add --active fastapi "uvicorn[standard]"
```

`--python ">=3.11"` обязателен по той же причине, что у Poetry: без него uv может записать в `requires-python` версию Python со своего ноутбука, и у ревьюера с более старой версией `uv sync` сломает окружение. `--active` ставит пакеты в ваш активный `venv/`, а не в новую папку `.venv/`: приложение запускается той же командой, что и у остальных команд.

**Ожидаемый вывод:** `Initialized project ...`, затем список строк `+ пакет==версия`, среди них `+ fastapi==` и `+ uvicorn==`; появились файлы `pyproject.toml` (в нём строка `requires-python = ">=3.11"`) и `uv.lock`.

**К2. Создать `main.py`** с содержимым:

```python
from fastapi import FastAPI

app = FastAPI(
    title="Project Name",
    description="Project description",
    version="0.1.0"
)


@app.get("/")
async def root():
    """Root endpoint."""
    return {"message": "Hello World"}


@app.get("/health")
async def health_check():
    """Health check endpoint."""
    return {"status": "healthy"}
```

Код оставьте как есть: на нём построены следующие практики.

**К3. Проверить приложение до коммита:**

```bash
uvicorn main:app --reload
```

**Ожидаемый вывод:** строка `Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)`.

Откройте второе окно терминала (окружение в нём активировать не нужно) и проверьте эндпоинт:

macOS / Linux:

```bash
curl http://localhost:8000/health
```

Windows PowerShell (`curl` в PowerShell — другая команда, поэтому `curl.exe`):

```powershell
curl.exe http://localhost:8000/health
```

**Ожидаемый вывод:** `{"status":"healthy"}`. В браузере: `http://localhost:8000` — `{"message":"Hello World"}`, `http://localhost:8000/docs` — страница Swagger UI с заголовком `Project Name`. Остановите сервер: `Ctrl+C` в первом окне.

**Полезные ссылки:**

- [FastAPI First Steps](https://fastapi.tiangolo.com/tutorial/first-steps/)
- [Poetry Documentation](https://python-poetry.org/docs/)
- [uv Documentation](https://docs.astral.sh/uv/)

##### Задача А: раздел «API» в `README.md`

Допишите в **самый конец** `README.md`, после раздела «Установка и запуск», раздел:

```markdown
## API

| Метод и путь | Ответ | Назначение |
|---|---|---|
| `GET /` | `{"message": "Hello World"}` | Проверка, что приложение отвечает |
| `GET /health` | `{"status": "healthy"}` | Health-check: приложение живо |
| `GET /docs` | Страница Swagger UI | Документация API, FastAPI генерирует её автоматически |
```

Ответы сверьте с кодом из PR задачи К, когда он появится. Остальные разделы README не меняйте.

##### Задача Б: раздел «Предметная область» в `README.md`

Допишите в **самый конец** `README.md`, после раздела «Установка и запуск», раздел по идее, выбранной на шаге 1.1. Строки с `ЗАПОЛНИТЬ:` замените целиком:

```markdown
## Предметная область

- **Основная сущность:** ЗАПОЛНИТЬ: например, «заявка на ремонт оборудования»
- **Пользователи:** ЗАПОЛНИТЬ: кто работает с приложением и зачем
- **Планируемые эндпоинты:**
  - ЗАПОЛНИТЬ: создание, например `POST /tickets`
  - ЗАПОЛНИТЬ: список, например `GET /tickets`
  - ЗАПОЛНИТЬ: одна запись, например `GET /tickets/{id}`
  - ЗАПОЛНИТЬ: изменение, например `PUT /tickets/{id}`
  - ЗАПОЛНИТЬ: удаление, например `DELETE /tickets/{id}`
```

Пять эндпоинтов — это CRUD из MVP [технического задания](../../project-requirements.md). Остальные разделы README не меняйте.

##### Задача В (команда из 4 человек): чек-лист ревьюера в `CONTRIBUTING.md`

В `CONTRIBUTING.md` в конец раздела «Code Review» (перед разделом «Встречи команды») добавьте подраздел:

```markdown
### Что ревьюер делает перед approval

1. Читает изменения во вкладке Files changed
2. Если PR меняет код или зависимости — переключается на ветку PR, ставит зависимости, запускает приложение и проверяет `GET /health`
3. Проверяет, что сообщения коммитов следуют Conventional Commits
4. Проверяет, что в PR нет `venv/`, `__pycache__/`, `.env`
5. Ставит Approve только после пунктов 1–4; замечания оставляет комментарием к конкретной строке
```

Не меняйте названия существующих разделов.

**Что должно получиться (все задачи):** `git status` показывает изменёнными или новыми только файлы вашей задачи, `venv/` в списке нет.

#### Шаг 3.3: Коммит, push и Pull Request — каждый участник (15 мин)

**1. Коммит.** Выберите строку `git add` по своей задаче:

```bash
# Задача К, вариант pip
git add main.py requirements.txt
# Задача К, вариант Poetry
git add main.py pyproject.toml poetry.lock
# Задача К, вариант uv
git add main.py pyproject.toml uv.lock
# Задачи А и Б
git add README.md
# Задача В
git add CONTRIBUTING.md
```

Сообщение коммита:

| Задача | Команда |
|---|---|
| К | `git commit -m "feat: add initial FastAPI application with health check"` |
| А | `git commit -m "docs: describe API endpoints in README"` |
| Б | `git commit -m "docs: describe project domain in README"` |
| В | `git commit -m "docs: add reviewer checklist to CONTRIBUTING"` |

**Как проверить:** `git status` выводит `nothing to commit, working tree clean`. Если остались неотслеживаемые файлы вашей задачи — вы выбрали не ту строку `git add`: добавьте их и сделайте `git commit --amend --no-edit`.

**2. Push:**

```bash
git push -u origin feature/<ваша-задача>
```

**Ожидаемый вывод:**

```text
remote: Create a pull request for 'feature/<ваша-задача>' on GitHub by visiting:
remote:      https://github.com/<логин-владельца>/app-conf-project-<тема>/pull/new/feature/<ваша-задача>
...
 * [new branch]      feature/<ваша-задача> -> feature/<ваша-задача>
```

**3. Pull Request.** Откройте ссылку из вывода `git push` (или в репозитории: Pull requests → New pull request → base: `main`, compare: ваша ветка) и заполните:

- **Title:** сообщение вашего коммита
- **Description:** что добавлено и как проверить. Для задачи К — какими командами поставить зависимости и запустить приложение
- **Reviewers** (шестерёнка справа): ревьюер по таблице ролей из шага 1.1

Нажмите **Create pull request**. Не мержите PR до approval ревьюера.

**Если вносите изменение в веб-редакторе GitHub** (нет своего ноутбука): в окне **Commit changes** замените автоматическое сообщение `Update README.md` на сообщение по Conventional Commits из таблицы выше, выберите **Create a new branch for this commit and start a pull request** и впишите имя ветки из шага 3.1 вместо предложенного `<логин>-patch-1`.

**Что должно получиться:** во вкладке Pull requests — открытый PR от каждого участника, автор PR — его аккаунт, у каждого PR назначен ревьюер.

**Полезные ссылки:**

- [Creating a pull request (GitHub Docs)](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/creating-a-pull-request)

**Точка синхронизации 120.**

#### Шаг 3.4: Ревью и merge (15 мин)

**Что делает ревьюер.**

1. Открыть PR → вкладка **Files changed**, прочитать изменения.
2. **PR задачи К — запустить код на своём ноутбуке.** Окружение активно, ваша собственная работа закоммичена:

```bash
git fetch origin
git switch feature/initial-setup
```

Установить зависимости — командой того менеджера, который выбрал автор:

```bash
# pip
pip install -r requirements.txt
# Poetry
pip install poetry
poetry install --no-root
# uv
pip install uv
uv sync --active
```

Затем `uvicorn main:app --reload` и проверка `/health`, как в задаче К (пункт К3). После проверки вернитесь в свою ветку: `git switch feature/<ваша-задача>`.

Если `pip install -r requirements.txt` падает на пакете `uvloop` — файл создан на macOS или Linux, а вы на Windows: `uvloop` под Windows не ставится. Поставьте зависимости напрямую — `pip install fastapi "uvicorn[standard]"` — и напишите об этом комментарий в PR. Это наблюдение понадобится на воркшопе №3, когда команда будет выбирать менеджер зависимостей.

3. **PR задачи А** — сверить ответы в таблице с `main.py` из PR задачи К. **PR задачи Б** — сверить с идеей, выбранной на шаге 1.1. **PR задачи В** — каждый пункт чек-листа проверяем, а не «код хороший».
4. **Review changes** → **Approve** → **Submit review**. Если есть замечания — **Request changes** с комментарием к строке; автор исправляет, коммитит и пушит в ту же ветку, вы смотрите ещё раз.

Approve ставится из **своего** аккаунта. Автору PR кнопка Approve недоступна — это правило GitHub, его не обойти.

**Что делает автор после approval:** **Merge pull request** → **Confirm merge**.

**Что должно получиться:** у PR статус **Merged**; на вкладке Conversation запись `<ревьюер> approved these changes` стоит **выше** записи о merge.

**Полезные ссылки:**

- [About pull request reviews (GitHub Docs)](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/reviewing-changes-in-pull-requests/about-pull-request-reviews)

#### Шаг 3.5: Разрешить конфликт (15 мин)

Выполняет автор того из PR А и Б, который мержится **вторым**. На странице его PR GitHub пишет `This branch has conflicts that must be resolved`, кнопка merge неактивна: обе ветки дописали текст в одно и то же место `README.md`, и Git не знает, в каком порядке его оставить.

```bash
git switch feature/<ваша-задача>
git fetch origin
git merge origin/main
```

**Ожидаемый вывод:**

```text
Auto-merging README.md
CONFLICT (content): Merge conflict in README.md
Automatic merge failed; fix conflicts and then commit the result.
```

Откройте `README.md`. В конце файла — блок:

```text
<<<<<<< HEAD
## Предметная область
...ваш раздел...
=======
## API
...раздел из main...
>>>>>>> origin/main
```

Нужны **оба** раздела. Удалите три строки-маркера (`<<<<<<< HEAD`, `=======`, `>>>>>>> origin/main`) и оставьте разделы друг за другом, разделив пустой строкой. В VS Code то же самое делает кнопка **Accept Both Changes** над блоком; после неё вставьте пустую строку между разделами — кнопка склеивает их подряд.

**Как проверить, что маркеров не осталось:**

macOS / Linux:

```bash
grep -n -E "^(<<<<<<<|=======|>>>>>>>)" README.md
```

Windows PowerShell:

```powershell
Select-String -Pattern "^(<<<<<<<|=======|>>>>>>>)" README.md
```

**Ожидаемый вывод:** пусто. Затем:

```bash
git add README.md
git commit -m "chore: merge main into feature/<ваша-задача>"
git push
```

Сообщение задавайте через `-m`: без него Git откроет текстовый редактор. Если он всё же открылся и внизу экрана видны `~` — это Vim: наберите `:wq` и нажмите Enter.

**Без терминала (веб-редактор GitHub).** На странице PR нажмите **Resolve conflicts**. В редакторе удалите три строки-маркера и оставьте оба раздела → **Mark as resolved** → **Commit merge**. Сообщение `Merge branch 'main' into feature/<ваша-задача>` GitHub подставит сам — для этого коммита оно допустимо.

**Что должно получиться:** на странице PR вместо сообщения о конфликте — `This branch has no conflicts with the base branch`. Ревьюер смотрит новый коммит во вкладке Files changed: в README должны быть оба раздела. Повторный Approve не нужен — прежний остаётся в силе, пока защита ветки не включена (практика №2). Если раздел потерян — Request changes. Затем автор мержит.

**Если конфликта не возникло** — кто-то из авторов вставил раздел не в конец файла. Конфликт всё равно нужно отработать: участники А и Б создают от обновлённого `main` по новой ветке (`feature/status-a`, `feature/status-b`), каждый меняет одну и ту же строку `Проект в разработке.` в разделе «Статус» на свой вариант, оба открывают PR; первый мержится, второй разрешает конфликт по этому шагу.

**Полезные ссылки:**

- [Resolving a merge conflict using the command line (GitHub Docs)](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/addressing-merge-conflicts/resolving-a-merge-conflict-using-the-command-line)

#### Шаг 3.6: Синхронизировать `main` и проверить результат — каждый участник (5 мин)

```bash
git switch main
git pull origin main
git log --oneline --graph -12
```

**Ожидаемый вывод:** в графе видны коммиты `Merge pull request #N from <логин-владельца>/feature/...` для каждого PR и merge-коммит, которым разрешён конфликт (`chore: merge main into ...` или `Merge branch 'main' into ...`); `git status` выводит `Your branch is up to date with 'origin/main'`.

**Ревьюер PR задачи К** (у него зависимости уже установлены на шаге 3.4) запускает приложение из `main` (`uvicorn main:app --reload`) и проверяет `/health`. Если запускает другой участник — сначала он ставит зависимости той же командой, что в шаге 3.4.

**Как проверить вклад каждого:**

```bash
git shortlog -sn HEAD
```

**Ожидаемый вывод:** в списке авторов — каждый участник команды под своим именем. Число учитывает и merge-коммиты, поэтому у того, кто мержил и разрешал конфликт, оно больше — это нормально.

**Точка синхронизации 155.**

#### Если время кончается

Нельзя переносить: приглашения коллабораторов, первый коммит, открытые PR каждого участника — без них нечего доводить в домашнем задании и не виден вклад каждого. Можно перенести в домашнее задание (задание 1): ревью и merge оставшихся PR и разрешение конфликта. Сообщите преподавателю на этапе 4, что именно осталось.

#### Если вы закончили раньше (бонусные задачи)

1. **Лицензия через веб-интерфейс.** В репозитории: Add file → Create new file → имя `LICENSE` → **Choose a license template** → MIT → Review and submit. В окне Commit changes: сообщение `docs: add MIT license`, **Create a new branch** с именем `feature/license`, затем PR с ревью. После merge добавьте в README раздел «Лицензия» — отдельным PR
2. **Цикл Request changes.** Ревьюер одного из PR сначала ставит Request changes с конкретным замечанием к строке, автор исправляет и пушит в ту же ветку, ревьюер ставит Approve. В PR должна остаться видна вся цепочка
3. **Разбор графа.** Найдите в `git log --oneline --graph` коммит, которым разрешён конфликт, и объясните в комментарии к PR, почему у него два родителя

---

### Этап 4: Самопроверка и фиксация (15 мин)

#### Шаг 4.1: Самопроверка по чек-листу (10 мин)

Команда проходит [чек-лист самопроверки](#чек-лист-самопроверки-для-команды) и отмечает пункты. Незакрытые пункты записываются — они уйдут в домашнее задание.

#### Шаг 4.2: Отчёт преподавателю (5 мин)

Одно сообщение в чат курса (шаблон ниже). Если преподаватель подходит к команде в это время — покажите работающее приложение из `main` (`/health`) и список PR, назовите незакрытые пункты.

```text
Команда: <название команды; если отдельного названия нет — имена участников>
Тема: <название проекта>
Репозиторий: https://github.com/<логин-владельца>/app-conf-project-<тема>
Pull requests: https://github.com/<логин-владельца>/app-conf-project-<тема>/pulls?q=is%3Apr
Менеджер зависимостей: pip / Poetry / uv
Не закрыто: <пункты чек-листа или «всё закрыто»>
```

---

### Буфер (10 мин)

Резерв на установку Python и Git, вход в GitHub из терминала, неотвеченные приглашения, разрешение конфликта. Если буфер не понадобился — берите бонусные задачи этапа 3.

---

## Если не работает

| Симптом | Причина | Что сделать |
|---|---|---|
| Windows: `python` открывает Microsoft Store или пишет `Python was not found` | Python не установлен или отключены псевдонимы команд | Пуск → «Manage app execution aliases» (Параметры → Приложения → Дополнительные параметры → Псевдонимы выполнения приложений): включить псевдонимы Python (default) / Python install manager, выключить псевдонимы App Installer; если уже включены — выключить и включить снова; открыть новое окно PowerShell. Python не установлен — Python install manager с python.org, затем `py install 3.14` |
| macOS: `python3 --version` выводит `3.9.x` | Это Python из Xcode Command Line Tools | Установить Python 3.11+ с python.org и открыть новое окно терминала |
| PowerShell: `running scripts is disabled on this system` / `выполнение сценариев отключено в этой системе` | Политика выполнения запрещает скрипты | `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned`, ответить `Y`, повторить активацию (шаг 2.1) |
| Linux: `ensurepip is not available` при `python3 -m venv venv` | Не установлен пакет venv | Debian / Ubuntu: `sudo apt install python3-venv`, затем удалить неполную папку `venv` и повторить |
| Git Bash в Windows: `venv/bin/activate: No such file or directory` | В Windows папка окружения называется `venv/Scripts` | `source venv/Scripts/activate` |
| macOS / Linux: `python: command not found` вне окружения | В системе есть только `python3` | Вне окружения используйте `python3`; внутри активного `venv` работает `python` |
| `zsh: no matches found: uvicorn[standard]` | zsh раскрывает квадратные скобки | Взять в кавычки: `"uvicorn[standard]"` |
| `Author identity unknown` / `Please tell me who you are` | Не задано имя и email | «Настройка Git и вход в GitHub» в «Подготовке», пункт 1, затем повторить `git commit` |
| `Support for password authentication was removed` | Введён пароль от аккаунта | «Настройка Git и вход в GitHub» в «Подготовке», пункт 2: Git Credential Manager, `gh auth login` или токен |
| `Permission to ... denied to <логин>` или ошибка 403 при push | Участник не коллаборатор или не принял приглашение | Шаг 1.2: владелец проверяет Collaborators, участник принимает приглашение |
| `GH007: Your push would publish a private email address` | В `user.email` указан скрытый адрес | Указать адрес `...@users.noreply.github.com` («Настройка Git и вход в GitHub» в «Подготовке»), затем `git commit --amend --reset-author --no-edit` и снова push |
| `error: src refspec main does not match any` | Коммита ещё нет или локальная ветка называется `master` | `git log` — есть ли коммит; `git branch --show-current` — если `master`, `git branch -M main` |
| `fatal: pathspec 'requirements.txt' did not match any files` | Команда работает на Poetry или uv | Строка `git add` для своего менеджера (шаг 3.3) |
| GitHub показывает `requirements.txt` как `Binary file not shown` | Файл записан в UTF-16 через `>` в PowerShell | Пересоздать: `pip freeze \| Out-File -Encoding utf8 requirements.txt`, закоммитить `fix: re-encode requirements.txt as UTF-8` |
| `pip install -r requirements.txt` падает на `uvloop` в Windows | Файл снят на macOS или Linux | Шаг 3.4: поставить `fastapi` и `"uvicorn[standard]"` напрямую, отметить в PR |
| `uvicorn: command not found` или `uvicorn is not recognized` при активном `venv` | В вашем окружении не установлены зависимости проекта | Поставить командой своего менеджера из шага 3.4 (`pip install -r requirements.txt`, `poetry install --no-root`, `uv sync --active`) и повторить |
| `poetry` / `uv`: `command not found` или `is not recognized` | Окружение `venv` не активировано | Активировать (шаг 2.1) и повторить |
| `uv sync --active` пишет `Access is denied`, окружение `venv` сломано или пересоздано на другой версии Python | В `pyproject.toml` `requires-python` выше вашей версии Python: `uv init` выполнен без `--python ">=3.11"` | Автор PR задачи К меняет строку на `requires-python = ">=3.11"`, выполняет `uv lock`, коммитит `pyproject.toml` и `uv.lock` в ту же ветку. Ревьюер удаляет папку `venv`, создаёт её заново (шаг 2.1) и повторяет шаг 3.4 |
| `poetry install`: `If you do not want to install the current project use --no-root` | Проект не является пакетом | `poetry install --no-root` |
| `Address already in use` (Windows: `[WinError 10048] Only one usage of each socket address`) при запуске uvicorn | Порт 8000 занят | `uvicorn main:app --reload --port 8001` и проверять `http://localhost:8001` |
| Сокомандника нет в списке Reviewers | Он не коллаборатор или не принял приглашение | Шаг 1.2 |
| Кнопки Approve нет | Вы автор этого PR | Approve ставит другой участник из своего аккаунта |
| `This branch has conflicts that must be resolved` | Две ветки изменили одно место файла | Шаг 3.5 |
| `fatal: Need to specify how to reconcile divergent branches` при `git pull origin main` | Вы закоммитили прямо в локальный `main` | Перенести коммит в ветку: `git switch -c feature/<задача>`, затем `git switch main` и `git reset --hard origin/main` |
| Заголовки README на GitHub отображаются как `\#` | Файл набран в визуальном редакторе, который экранирует разметку | Открыть файл в VS Code, удалить обратные косые перед `#`, `-`, `*`, закоммитить исправление |
| Windows, при `git add`: `warning: in the working copy of '...', LF will be replaced by CRLF` | Git for Windows сообщает о преобразовании переводов строк | Это предупреждение, а не ошибка: ничего делать не нужно |
| На GitHub оказались файлы репозитория курса, а не ваши | Склонирован не тот репозиторий | `git remote -v` должен показывать ваш репозиторий; склонировать заново по шагу 1.4 |

---

## Сводка полезных ссылок

### Git и GitHub

- [GitHub Docs](https://docs.github.com/)
- [About Git (GitHub Docs)](https://docs.github.com/en/get-started/using-git/about-git)
- [Inviting collaborators to a personal repository](https://docs.github.com/en/account-and-profile/setting-up-and-managing-your-personal-account-on-github/managing-access-to-your-personal-repositories/inviting-collaborators-to-a-personal-repository)
- [Setting your commit email address](https://docs.github.com/en/account-and-profile/setting-up-and-managing-your-personal-account-on-github/managing-email-preferences/setting-your-commit-email-address)
- [GitHub CLI](https://cli.github.com/)
- [Resolving a merge conflict using the command line](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/addressing-merge-conflicts/resolving-a-merge-conflict-using-the-command-line)
- [Conventional Commits](https://www.conventionalcommits.org/)

### Python и FastAPI

- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [FastAPI First Steps](https://fastapi.tiangolo.com/tutorial/first-steps/)
- [Python Virtual Environments](https://docs.python.org/3/tutorial/venv.html)
- [Poetry Documentation](https://python-poetry.org/docs/)
- [uv Documentation](https://docs.astral.sh/uv/)

### Документы курса

- [Домашнее задание №1](../../home-work-1.md) — идеи проекта, из которых выбирается тема
- [Техническое задание](../../project-requirements.md) — MVP и примеры тем
- [Definition of Done](../../definition-of-done.md) — критерии готовности проекта
- [Система оценивания](../../assessment.md) — оценка 360°, вклад каждого участника

---

## Чек-лист самопроверки для команды

### Репозиторий и доступ

- [ ] Репозиторий публичный: открывается в окне инкогнито без входа в аккаунт
- [ ] Settings → Collaborators: все сокомандники без пометки «Pending invite», `andrey-limasov` приглашён
- [ ] У каждого участника `git config --global user.name` и `user.email` выводят его имя и email из GitHub

### Каркас

- [ ] `.gitignore` по шагу 2.2; папки `venv/` на GitHub нет
- [ ] `README.md`: название и описание выбранной темы, все участники с логинами, ваша версия Python, разделы «API» и «Предметная область»
- [ ] `CONTRIBUTING.md`: вместо стратегии — заглушка «выбирается на воркшопе №2 и фиксируется в ADR 001»; разделы «Встречи команды», «Разрешение конфликтов», «Контакты» на месте и заполнены
- [ ] Проверка на `ЗАПОЛНИТЬ` (шаг 2.5) по `README.md` и `CONTRIBUTING.md` выводит пусто

### Код и зависимости

- [ ] В `main` есть `main.py` с `GET /` и `GET /health`
- [ ] В `main` есть файлы зависимостей своего менеджера: pip — `requirements.txt` в UTF-8; Poetry — `pyproject.toml` и `poetry.lock`; uv — `pyproject.toml` и `uv.lock`
- [ ] Ревьюер PR задачи К поставил зависимости на своём ноутбуке и запустил приложение

### Git и Pull Request

- [ ] Первый коммит в `main` — по Conventional Commits
- [ ] У каждого участника есть свой смерженный PR, автор PR — его аккаунт
- [ ] Каждый PR получил Approve от участника, который не является автором, до merge
- [ ] Конфликт разрешён: в ветке второго из PR А и Б есть merge-коммит с двумя родителями (`chore: merge main into ...` или `Merge branch 'main' into ...`), в README оба раздела, маркеров конфликта нет
- [ ] Все сообщения коммитов следуют Conventional Commits (merge-коммиты, созданные кнопкой GitHub, не в счёт)
- [ ] `git shortlog -sn HEAD` показывает каждого участника под своим именем
- [ ] Локальный `main` у каждого совпадает с GitHub

### Работа приложения

- [ ] Из `main`: `uvicorn main:app --reload` запускается без ошибок
- [ ] `http://localhost:8000` отвечает `{"message":"Hello World"}`
- [ ] `http://localhost:8000/health` отвечает `{"status":"healthy"}`
- [ ] `http://localhost:8000/docs` показывает документацию OpenAPI

---

## Чек-лист для преподавателя

Сначала примите приглашения всех команд (github.com/notifications или письма GitHub): без этого вы не видите список коллабораторов. Проверки ниже выполняются по ссылкам из отчёта команды (шаг 4.2); в классе проверяется только работающее приложение. Пункты со звёздочкой — критические: без них работа не засчитывается.

| Что проверяем | Как проверяем | Признак выполнения |
|---|---|---|
| Репозиторий публичный * | Открыть ссылку в окне инкогнито | Открывается без входа |
| Преподаватель — коллаборатор * | Приглашение в github.com/notifications или на `https://github.com/<владелец>/<репозиторий>/invitations` | Приглашение пришло и принято |
| Все участники — коллабораторы * | После принятия своего приглашения: `gh api repos/<владелец>/<репозиторий>/collaborators --jq ".[].login"` | В списке логины всех участников и ваш; приглашения, которые ещё не приняты, в этот список не попадают |
| Тема выбрана | `README.md` в `main` | Название и описание темы, раздел «Предметная область» с сущностью и CRUD-эндпоинтами |
| Каркас | Файлы в `main` | `.gitignore` по шагу 2.2; в `CONTRIBUTING.md` — заглушка стратегии, разделы практики №1 на месте; слова `ЗАПОЛНИТЬ` нет |
| Зависимости | Файлы в `main`, история PR задачи К | pip — `requirements.txt`, GitHub показывает его текстом, а не `Binary file not shown`; Poetry / uv — `pyproject.toml` и lock-файл |
| PR каждого участника * | Pull requests → Closed | Смерженных PR не меньше, чем участников; у каждого участника хотя бы один PR, автор — его аккаунт |
| Approval до merge * | Вкладка Conversation каждого PR | `approved these changes` от участника, который не автор, выше записи о merge |
| Конфликт разрешён | Коммиты ветки второго из PR А и Б или `git log --oneline --graph` в клоне | Есть merge-коммит с двумя родителями (`chore: merge main into ...` или `Merge branch 'main' into ...` из веб-редактора); в README оба раздела, маркеров нет |
| Вклад виден в истории | `git shortlog -sn HEAD` в клоне репозитория | Каждый участник — отдельной строкой под своим именем, нет общего «аккаунта команды» |
| Приложение работает | На обходе этапа 4, на ноутбуке команды | `/health` отвечает `{"status":"healthy"}` из `main` |

**Что фиксировать в ведомости по каждой команде:** название команды · тема · ссылка на репозиторий · менеджер зависимостей (pip / Poetry / uv) · число участников и число участников со смерженным PR · критические пункты · пункты, перенесённые в домашнее задание.

---

## Связь с Definition of Done

Практика работает на следующие пункты [Definition of Done](../../definition-of-done.md).

### Раздел 1. Репозиторий и контроль версий

| Пункт DoD | Чем закрывается на этой практике |
|---|---|
| Публичный репозиторий на GitHub/GitLab | Шаг 1.2 |
| Осмысленные сообщения коммитов (Conventional Commits) | Шаги 2.5, 3.3, 3.5 — сообщения заданы явно |
| Нет закоммиченных `.env`, `__pycache__`, виртуального окружения | Шаг 2.2 — `.gitignore`; проверка `git status` в шагах 2.2 и 3.2 |
| Все изменения в `main` только через Pull Request | Этап 3 — все изменения после первого коммита идут через PR. Первый коммит (шаг 2.5) — единственное исключение: веток и PR без него не создать. Сервер начнёт требовать PR на практике №2 |
| Каждый PR имеет минимум 1 approval от другого участника команды | Шаг 3.4 |
| Есть файл `.gitignore` с правильными правилами | Шаг 2.2 |
| Есть файл `CONTRIBUTING.md` с правилами работы в команде | Шаг 2.4, задача В |

Пункт «Применена стратегия ветвления» на этой практике не закрывается: стратегию выбирают на воркшопе №2, применяют на практике №2.

### Раздел 6. Наблюдаемость

| Пункт DoD | Чем закрывается на этой практике |
|---|---|
| Есть health-check эндпоинт `GET /health` | Задача К. Проверка подключения к базе данных добавится на практике №3 |

### Раздел 7. Документация

| Пункт DoD | Чем закрывается на этой практике |
|---|---|
| `README.md`: название проекта и краткое описание | Шаг 2.3 |
| `README.md`: стек технологий | Шаг 2.3 |
| `README.md`: список участников команды | Шаг 2.3 |
| `README.md`: примеры API запросов | Задача А — частично: перечень эндпоинтов и ответов |
| `CONTRIBUTING.md`: правила написания коммитов | Шаг 2.4 |
| `CONTRIBUTING.md`: правила code review | Шаг 2.4, задача В |
| `CONTRIBUTING.md`: контакты команды | Шаг 2.4 |
| OpenAPI: автоматически генерируется FastAPI (`/docs`) | Задача К, пункт К3 |

---

## Связь с другими занятиями

| Занятие | Что берёт или передаёт |
|---|---|
| Воркшоп №1 и домашнее задание №1 | Команды и три идеи проекта — вход шага 1.1 |
| Воркшоп №2 | Выбор стратегии ветвления и ADR 001. Заглушка в разделе «Git-стратегия» `CONTRIBUTING.md` ждёт именно этого решения |
| Практика №2 | Берёт: коллабораторов с принятыми приглашениями, настроенный Git у каждого, `CONTRIBUTING.md` с разделами практики №1, историю с PR. Вписывает стратегию в `CONTRIBUTING.md` по ADR 001 и включает защиту `main` |
| Воркшоп №3 | Сравнивает менеджеры зависимостей. Ваш `requirements.txt` из `pip freeze` или `pyproject.toml` с lock-файлом — исходное состояние для ADR 002; наблюдение про `uvloop` из шага 3.4 — аргумент в этом сравнении |
| Практика №3 | Берёт `main.py` в корне, `GET /health`, окружение `venv/`, раздел README «Установка и запуск» |

---

## Домашнее задание

| № | Задание | Дедлайн |
|---|---|---|
| 1 | Закрыть незавершённое с практики | 3 дня после практики |
| 2 | Принять приглашения в репозиторий | 1 день после практики |
| 3 | Подготовиться к воркшопу №2 | К началу воркшопа №2 |

### 1. Закрыть незавершённое

Довести до merge всё, что не успели: PR, ревью, разрешение конфликта, пункты чек-листа самопроверки, названные на фиксации. Если у кого-то из участников ещё нет своего PR — он делает его по этапу 3 с задачей, которую не взяли на занятии (А, Б или В), или с бонусной задачей 1.

**Критерий успеха:**

- [ ] Нет открытых PR, созданных на практике
- [ ] У каждого участника есть смерженный PR с approval другого участника
- [ ] В истории есть разрешённый конфликт (шаг 3.5)
- [ ] Чек-лист самопроверки закрыт полностью

### 2. Принять приглашения

Каждый сокомандник, у которого в Settings → Collaborators ещё стоит «Pending invite», принимает приглашение. Приглашение действует 7 дней; истекшее владелец отправляет заново.

**Критерий успеха:**

- [ ] В Settings → Collaborators нет сокомандников с «Pending invite»

### 3. Подготовиться к воркшопу №2

Преподаватель после практики объявит в чате курса, какую стратегию ветвления представляет ваша команда. Изучите по ней [материалы для самостоятельного изучения](../../workshops/workshop2/workshop2-materials.md) и подготовьте презентацию по [шаблону](../../workshops/workshop2/workshop2-presentation-templates.md). Стратегию для своего проекта в `CONTRIBUTING.md` не вписывайте — её выберете на воркшопе.

**Критерий успеха:**

- [ ] Материалы по назначенной стратегии прочитаны всеми участниками
- [ ] Черновик презентации по шаблону готов к началу воркшопа

---

## Дополнительные материалы

- [Pro Git Book](https://git-scm.com/book/en/v2) — бесплатная книга о Git
- [FastAPI Full Tutorial](https://fastapi.tiangolo.com/tutorial/) — полное руководство по FastAPI
- [Real Python](https://realpython.com/) — статьи по Python

---

## История изменений

| Дата | Изменение |
|---|---|
| 2026-10-10 | Инструкция переработана по итогам проведённого занятия. Добавлены: выбор темы из идей ДЗ №1 (шаг 1.1) и таблица ролей; приглашение в коллабораторы всех участников, а не только преподавателя (шаг 1.2); настройка `user.name`/`user.email` и входа в GitHub из терминала (шаг 1.3); этап параллельной работы — у каждого участника своя ветка и PR, два PR правят `README.md`, второй разрешает конфликт (этап 3); локальная проверка приложения до PR и ревьюером, синхронизация `main` после merge. Ветки Poetry и uv доведены до коммита `pyproject.toml` и lock-файла. Команды для Windows PowerShell (политика выполнения, `Activate.ps1`, `Out-File -Encoding utf8`, `curl.exe`), macOS (`python3`, кавычки вокруг `"uvicorn[standard]"`) и Linux даны отдельно. В `CONTRIBUTING.md` стратегия ветвления заменена заглушкой «выбирается на воркшопе №2, фиксируется в ADR 001». Плейсхолдеры обозначены словом `ЗАПОЛНИТЬ:` с проверкой командой. Тайминг сведён к 180 минутам с буфером и точками синхронизации. Добавлены разделы «Если не работает», «Чек-лист для преподавателя», «Связь с Definition of Done», «Связь с другими занятиями», «Домашнее задание». GitLab убран: курс работает на GitHub. Уточнения после повторного ревью: тема не блокирует работу — преподаватель принимает её на обходе, при смене темы репозиторий переименовывается (шаг 1.1); для uv — `uv init --bare --python ">=3.11"`; веб-путь разрешения конфликта (шаг 3.5); установка Python в Windows через Python install manager и псевдонимы команд; новые строки в «Если не работает». После прогона с пустой машины: настройка Git и входа в GitHub вынесена в подготовку, на занятии — проверка (шаг 1.3); тайминг 35 + 40 + 80 + 15 + буфер 10; шаг 3.6 выполняет ревьюер PR задачи К; повторный Approve после разрешения конфликта не нужен; тимлид и Tech Lead связаны с таблицей ролей. После прогона p1-3: вход без `gh` на Linux (ссылка на установку gh, токен с `credential.helper cache`), активация в Git Bash на Windows, русский текст ошибки политики PowerShell, пустой вывод `git config` = не настроено, пустая строка после «Accept Both Changes», версия Python в README — минимальная в команде. 2026-10-11: формулировка о платформе согласована с DoD (GitHub в инструкциях, GitLab допускается) |
