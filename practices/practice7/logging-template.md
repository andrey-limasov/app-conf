# Шаблон: структурированные логи с Request ID (практика №7)

Код для этапа 3 [практики №7](practice-7-guide.md): строки лога приложения — JSON в stdout; в каждой строке время, уровень, сообщение, `request_id`, метод и путь запроса; `request_id` возвращается клиенту заголовком `X-Request-ID`. Библиотека — по п. 7 «Решения» ADR 006: structlog или loguru. Вывод у обеих одинаковый, поэтому команды проверки в инструкции общие.

Как пользоваться: возьмите разделы 1–2 и 5–6 целиком, из разделов 3–4 — один, по своей библиотеке. Модуль настроек — `pydantic-settings` или python-dotenv: где они пишутся по-разному, даны оба варианта.

**Что проверено при подготовке (2026-11-11).** Код ниже — на копиях проектов команд после практики №6, Windows 11:

| Библиотека | Проект | Результат |
|---|---|---|
| structlog 26.1.0 | uv, Python 3.14, fastapi 0.143.0, `pydantic-settings` | `ruff check` (при порядке импортов из раздела 5), `ruff format --check`, `mypy` чистые; 5 тестов практики №6 зелёные на PostgreSQL 16; `X-Request-ID` возвращается; строка из синхронного `/health` с `request_id` |
| structlog 26.1.0 | uv, Python 3.12, fastapi 0.142.2, `pydantic-settings` | То же (без запуска тестов) |
| structlog 26.1.0 | Poetry, Python 3.14, fastapi 0.143.0, python-dotenv | `poetry add` и `poetry export` — строка `structlog==26.1.0` в `requirements.txt`; линтеры чистые; `LOG_LEVEL=bogus` — остановка при старте |
| loguru 0.7.3 | uv, Python 3.12 и 3.14, `pydantic-settings` | Линтеры чистые (с пометкой `noqa` ниже); строки с `request_id`, в том числе из синхронного эндпоинта и с трассировкой ошибки |
| loguru 0.7.3 | Poetry 2.5.1, `requires-python = ">=3.11"`, python-dotenv (прогон инструкции, 2026-11-11) | `poetry add loguru==0.7.3` без `--python` — `version solving failed`; с `--python ">=3.11,<4.0"` — `Writing lock file`, строка `loguru==0.7.3 ; python_version >= "3.11" and python_version < "4.0"` в `requirements.txt`; линтеры чистые; локальная проверка раздела 7 — как у uv |

Образ с этим кодом за прокси Caddy, Nginx и Traefik — `X-Request-ID` проходит через прокси в обе стороны ([шаблон reverse-proxy](reverse-proxy-template.md)).

**Тесты проекта, которые читают stdout приложения**, после подключения модуля получат строки JSON в выводе — инструкция, шаг 3.5.

---

## 1. Зависимость

Версия — закреплённая: она же войдёт в «Последствия» ADR 006.

```bash
# uv
uv add structlog==26.1.0        # или: uv add loguru==0.7.3

# Poetry
poetry add structlog==26.1.0
# loguru: нижняя граница — как в requires-python вашего pyproject.toml
poetry add loguru==0.7.3 --python ">=3.11,<4.0"
poetry export -f requirements.txt --output requirements.txt
```

Poetry и loguru: у loguru в метаданных `requires-python <4.0`, а в проектах курса `requires-python = ">=3.11"` без верхней границы — `poetry add loguru==0.7.3` без `--python` останавливается с `version solving failed` и подсказкой `python = ">=3.11,<4.0"`. `--python` ограничивает этим диапазоном только зависимость loguru; в `pyproject.toml` и `requirements.txt` строка loguru — с маркером `python_version`. В образ loguru ставится, если Python образа в этом диапазоне.

**Ожидаемый вывод:** uv — список установленных пакетов с `+ structlog==26.1.0` (loguru — `+ loguru==0.7.3`, на Windows ещё `+ win32-setctime==...`: зависимость loguru только для Windows, в образ Linux не попадёт); Poetry — `Writing lock file`, `poetry export` без вывода. Poetry без `poetry export`: `Dockerfile` ставит зависимости из `requirements.txt`, и библиотеки в образе не будет — контейнер упадёт с `ModuleNotFoundError`.

**Как проверить:** `git status --short` — изменены `pyproject.toml` и lock-файл (Poetry — и `requirements.txt`).

---

## 2. Уровень логов в модуле настроек

`LOG_LEVEL` — новый параметр конфигурации: не секрет, поэтому с умолчанием `INFO`; недопустимое значение — остановка при старте, как у `APP_ENV` (fail fast практики №4).

**`pydantic-settings`** — поле в класс `Settings` (`settings.py`), рядом с другими несекретными параметрами:

```python
    # Уровень логов: не секрет, умолчание есть, недопустимое значение — ошибка при старте
    log_level: Literal["DEBUG", "INFO", "WARNING", "ERROR"] = "INFO"
```

`Literal` у проектов курса уже импортирован (`app_env`). Ошибка при старте с `LOG_LEVEL=bogus`: `log_level` и `Input should be 'DEBUG', 'INFO', 'WARNING' or 'ERROR'`.

**python-dotenv** — в конец `settings.py`:

```python
# Уровень логов: не секрет, умолчание есть, недопустимое значение — остановка при старте
LOG_LEVEL = os.environ.get("LOG_LEVEL", "INFO")
if LOG_LEVEL not in ("DEBUG", "INFO", "WARNING", "ERROR"):
    sys.exit(
        f"LOG_LEVEL должна быть DEBUG, INFO, WARNING или ERROR, получено: {LOG_LEVEL}"
    )
```

---

## 3. Модуль логов: structlog

Файл `app_logging.py` в корне проекта, рядом с `main.py` (имя не `logging.py`: оно закрыло бы стандартный модуль `logging`).

```python
"""Структурированные логи: JSON в stdout, request_id в каждой строке запроса."""

import logging
import re
import uuid
from collections.abc import Awaitable, Callable

import structlog
from fastapi import Request, Response
from fastapi.responses import JSONResponse

from settings import settings

# Значение X-Request-ID от клиента принимается, только если оно короткое и без пробелов
REQUEST_ID_RE = re.compile(r"[A-Za-z0-9._-]{1,64}")

structlog.configure(
    processors=[
        structlog.contextvars.merge_contextvars,  # request_id и контекст запроса
        structlog.processors.add_log_level,
        structlog.processors.TimeStamper(fmt="iso", utc=True),
        structlog.processors.format_exc_info,
        structlog.processors.JSONRenderer(ensure_ascii=False),
    ],
    wrapper_class=structlog.make_filtering_bound_logger(
        logging.getLevelNamesMapping()[settings.log_level]
    ),
)
log = structlog.get_logger()


async def request_context(
    request: Request, call_next: Callable[[Request], Awaitable[Response]]
) -> Response:
    """Middleware: request_id, метод и путь — в каждую строку лога этого запроса."""
    incoming = request.headers.get("x-request-id", "")
    request_id = incoming if REQUEST_ID_RE.fullmatch(incoming) else uuid.uuid4().hex
    structlog.contextvars.clear_contextvars()
    structlog.contextvars.bind_contextvars(
        request_id=request_id, method=request.method, path=request.url.path
    )
    try:
        response = await call_next(request)
    except Exception:
        log.exception("request failed")
        response = JSONResponse(
            status_code=500, content={"detail": "Internal Server Error"}
        )
    response.headers["X-Request-ID"] = request_id
    log.info("request finished", status=response.status_code)
    return response
```

**python-dotenv:** строка `from settings import settings` → `import settings`, `settings.log_level` → `settings.LOG_LEVEL`.

Что здесь происходит — [structlog: Context Variables](https://www.structlog.org/en/stable/contextvars.html) (прочитано к воркшопу №7): `merge_contextvars` — первым процессором, `clear_contextvars` — в начале каждого запроса, значения — через `bind_contextvars`. Контекст привязывается в асинхронном middleware, поэтому он есть и в строках из синхронных эндпоинтов, как `/health`.

---

## 4. Модуль логов: loguru

Файл `app_logging.py` в корне проекта, рядом с `main.py`. Встроенный JSON loguru (`serialize=True`) кладёт поля во вложенный объект `record` с десятком служебных полей, поэтому здесь своя функция вывода: строка JSON с теми же полями, что у structlog.

```python
"""Структурированные логи: JSON в stdout, request_id в каждой строке запроса."""

import json
import re
import sys
import traceback
import uuid
from collections.abc import Awaitable, Callable
from datetime import UTC
from typing import TYPE_CHECKING

from fastapi import Request, Response
from fastapi.responses import JSONResponse
from loguru import logger as log

from settings import settings

if TYPE_CHECKING:
    from loguru import Message

# Значение X-Request-ID от клиента принимается, только если оно короткое и без пробелов
REQUEST_ID_RE = re.compile(r"[A-Za-z0-9._-]{1,64}")


def _json_sink(message: "Message") -> None:
    """Одна строка JSON на запись: время, уровень, сообщение и контекст."""
    record = message.record
    entry: dict[str, object] = {
        "timestamp": record["time"].astimezone(UTC).isoformat(),
        "level": record["level"].name.lower(),
        "event": record["message"],
        **record["extra"],  # request_id, method, path и поля из bind()
    }
    if record["exception"] is not None:
        exc_type, exc_value, exc_tb = record["exception"]
        entry["exception"] = "".join(
            traceback.format_exception(exc_type, exc_value, exc_tb)
        )
    sys.stdout.write(json.dumps(entry, ensure_ascii=False, default=str) + "\n")
    sys.stdout.flush()


log.remove()  # убрать обработчик по умолчанию (текст в stderr)
log.add(_json_sink, level=settings.log_level)


async def request_context(
    request: Request, call_next: Callable[[Request], Awaitable[Response]]
) -> Response:
    """Middleware: request_id, метод и путь — в каждую строку лога этого запроса."""
    incoming = request.headers.get("x-request-id", "")
    request_id = incoming if REQUEST_ID_RE.fullmatch(incoming) else uuid.uuid4().hex
    with log.contextualize(
        request_id=request_id, method=request.method, path=request.url.path
    ):
        try:
            response = await call_next(request)
        except Exception:  # noqa: BLE001 — любая ошибка: строка в лог и ответ 500
            log.exception("request failed")
            response = JSONResponse(
                status_code=500, content={"detail": "Internal Server Error"}
            )
        response.headers["X-Request-ID"] = request_id
        log.bind(status=response.status_code).info("request finished")
    return response
```

**python-dotenv:** строка `from settings import settings` → `import settings`, `settings.log_level` → `settings.LOG_LEVEL`.

`# noqa: BLE001` нужен: ruff 0.17.1 с настройками по умолчанию на этот `except Exception` с `log.exception` loguru выдаёт `BLE001` (на варианте structlog — нет, проверено). Документация: [loguru: Logger](https://loguru.readthedocs.io/en/stable/api/logger.html) — `contextualize()`, `bind()`, `add()` с функцией вместо файла (прочитано к воркшопу №7).

---

## 5. Подключить в `main.py`

Четыре правки (пути — для `main.py` в корне; у пакета `app/` — `app/app_logging.py` и импорт `from app.app_logging import ...`):

1. Удалить строку `import logging` (и пустую строку после неё, если она была первой) и строку `logger = logging.getLogger("uvicorn.error")`
2. Добавить импорт `from app_logging import log, request_context` среди импортов модулей проекта, по алфавиту: `pydantic-settings` — строкой **перед** `from settings import settings`; python-dotenv — после `import settings` (строки `import ...` идут перед `from ... import ...`). Другой порядок ruff 0.17.1 с настройками по умолчанию отмечает `I001 Import block is un-sorted or un-formatted` — исправляет `uv run ruff check --fix .`
3. Сразу после блока `app = FastAPI(...)` — строку регистрации middleware: `app.middleware("http")(request_context)` (то же, что декоратор `@app.middleware("http")` из [FastAPI: Middleware](https://fastapi.tiangolo.com/tutorial/middleware/), раздел **Create a middleware**, 3 мин)
4. В `/health` строку `logger.warning("Health check: database is unavailable: %s", exc)` заменить:

```python
        # structlog
        log.warning("database unavailable", error=str(exc))
        # loguru
        log.bind(error=str(exc)).warning("database unavailable")
```

Оставьте одну из двух строк. Другие вызовы `logger.` в коде — так же: сообщение без подстановок `%s`, значения — именованными полями.

**Пользователь в контексте** (DoD 6 «какой пользователь») — только если в проекте есть авторизация: добавьте поле в `bind_contextvars(...)` (structlog) или `contextualize(...)` (loguru). Авторизации нет — в п. 7 ADR 006 так и запишите.

**ADR 006, п. 8, вариант (в)** (`/docs` в `prod` открыты) — в `FastAPI(...)` три строки `docs_url=`, `redoc_url=`, `openapi_url=` удалите (и комментарий над ними, если он про «только в dev»): по умолчанию FastAPI их включает. Причина — строкой «Истории изменений» ADR 003 (инструкция, шаг 5.1).

---

## 6. `Dockerfile`: без access-строк uvicorn

В последней строке `CMD` добавить `"--no-access-log"`:

```dockerfile
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000", "--no-access-log"]
```

Без этого на каждый запрос uvicorn пишет ещё и текстовую строку `INFO:     172.18.0.4:51234 - "GET /health HTTP/1.1" 200 OK` — без `request_id`; её заменяет строка `request finished` из middleware. Строки запуска uvicorn (`INFO:     Started server process [1]`) остаются текстом: их пишет сам сервер. Параметры — `uv run uvicorn --help` (Poetry — `poetry run uvicorn --help`), строка `--access-log / --no-access-log`.

Миграции командой контейнера `app` (`command:` с `alembic upgrade head && uvicorn ...`): на сервере `--no-access-log` уже стоит в `command:` у `app` в `deploy/compose.yaml` ([шаблон reverse-proxy](reverse-proxy-template.md), раздел 7). Корневой `compose.yaml` не меняйте: на ноутбуке access-строки не мешают.

---

## 7. Как проверить локально

Без базы: `/health` отвечает `503`, и строка о недоступной базе приходит с тем `request_id`, который вы передали. Нужны два окна терминала в корне проекта. Адрес `127.0.0.1:1` — порт, на котором базы заведомо нет.

Окно 1 — запустить приложение:

```bash
# macOS / Linux / Git Bash; Poetry — poetry run вместо uv run
DATABASE_URL=postgresql://127.0.0.1:1/postgres uv run uvicorn main:app --port 8000 --no-access-log
```

```powershell
# Windows PowerShell; Poetry — poetry run вместо uv run
$env:DATABASE_URL = "postgresql://127.0.0.1:1/postgres"
uv run uvicorn main:app --port 8000 --no-access-log
```

Окно 2 — запрос с заголовком:

```bash
# macOS / Linux / Git Bash
curl -sS -i -H "X-Request-ID: local-1" http://127.0.0.1:8000/health
```

```powershell
# Windows PowerShell 5.1 и 7: curl.exe, а не curl
curl.exe -sS -i -H "X-Request-ID: local-1" http://127.0.0.1:8000/health
```

**Ожидаемый вывод.** Окно 2: `HTTP/1.1 503 Service Unavailable`, среди заголовков `x-request-id: local-1`, тело `{"status":"unhealthy",...}` (у вашего кода — свои поля). Окно 1, через 0–3 секунды (на Windows подключение к закрытому порту ждёт `connect_timeout`): две строки JSON — `"event": "database unavailable"` с полем `error` и `"event": "request finished"` со `"status": 503`, у обеих `"request_id": "local-1"`, `"method": "GET"`, `"path": "/health"`, `"level"`, `"timestamp"`. Остановить — `Ctrl+C` в окне 1; PowerShell — затем `Remove-Item Env:DATABASE_URL`.

Пример строки structlog (локальный прогон 2026-11-11, Windows; на Linux в `error` — `connection refused` или похожее):

```text
{"error": "connection timeout expired", "event": "database unavailable", "method": "GET", "path": "/health", "request_id": "local-1", "level": "warning", "timestamp": "2026-11-11T06:04:21.455913Z"}
```

loguru — те же поля в другом порядке, время с `+00:00` вместо `Z`.

Запрос без заголовка или с недопустимым значением (`X-Request-ID: bad id with spaces`) — в ответе и в логе 32 шестнадцатеричных символа: `request_id` сгенерирован.

---

## 8. Как смотреть логи на сервере

В папке `deploy` на сервере:

```bash
docker compose logs --no-log-prefix app | grep '"request_id": "p7-1"'      # строки одного запроса
docker compose logs --no-log-prefix --since 10m app | grep '"level": "error"'   # ошибки за 10 минут
docker compose logs --no-log-prefix -f app | grep -v '"path": "/health"'     # поток без проверок HEALTHCHECK
```

Каждые 10 секунд (интервал `HEALTHCHECK` в `Dockerfile` проектов курса) в лог приходит строка `request finished` с `"path": "/health"` — это проверка здоровья контейнера, а не пользователь. Это и есть «базовый мониторинг — просмотр логов на сервере» (DoD 6) — команды просмотра запишите в п. 7 ADR 006.

---

## 9. Хранение логов дольше жизни контейнера (домашнее задание 2)

Драйвер `local` из `deploy/compose.yaml` хранит строки, пока жив контейнер: при деплое нового тега `app` пересоздаётся, и строки прошлой версии пропадают (показано в демо воркшопа №7). Варианты, которые закрывают это (выбор и обоснование — в п. 7 ADR 006):

| Вариант | Что поменять | Как смотреть |
|---|---|---|
| Драйвер `journald`: строки — в системном журнале сервера | `deploy/compose.yaml`, у `app`: `logging:` → `driver: journald`, блок `options` удалить | `journalctl CONTAINER_NAME=<имя контейнера> --since today` (имя — `docker compose ps`); нужна группа `systemd-journal` — пользователя команды в неё добавляет курс; на Ubuntu хватает и группы `adm`. Проверка — `groups` на сервере, нет ни одной из двух — преподавателю ([Docker Docs: Journald logging driver](https://docs.docker.com/engine/logging/drivers/journald/), 3 мин) |
| Файл в томе: приложение пишет ещё и в файл | Второй вывод в `app_logging.py` (loguru — `log.add("/logs/app.log", rotation=..., retention=...)`), том `/logs` у `app`, каталог с правами пользователя образа | `docker compose exec app tail /logs/app.log` |

Драйвер `journald` проверен на сервере курса (2026-11-14, Ubuntu 24.04, доступ через группу `adm`): после пересоздания `app` строки прошлого контейнера находятся `journalctl`, `docker compose logs app` с этим драйвером тоже работает. Файл в томе курсом не проверен. **Как проверить:** после `docker compose up -d --force-recreate app` строки, записанные до пересоздания, находятся командой из п. 7.

---

## История изменений

| Дата | Что изменилось | Что это значит для команд |
|---|---|---|
| 2026-11-11 | Первая версия | — |
