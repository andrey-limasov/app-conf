# Практика №1: "Инфраструктура команды и первый коммит"

**Длительность:** 4 академических часа (180 минут)  
**Формат:** Самостоятельная командная работа по письменной инструкции  
**Роль преподавателя:** Организационная поддержка (проверка команд, фиксация прогресса)

---

## Цели практики

К концу занятия каждая команда должна:

1. **Создать** публичный репозиторий на GitHub/GitLab
2. **Добавить** преподавателя (`andrey-limasov`) как контрибьютора
3. **Настроить** базовую структуру проекта (`.gitignore`, `README.md`, `CONTRIBUTING.md`)
4. **Сделать** первый осмысленный коммит по правилам Conventional Commits
5. **Создать** минимальное FastAPI-приложение с health-check эндпоинтом
6. **Настроить** виртуальное окружение и зафиксировать зависимости
7. **Создать** feature-ветку и Pull Request

---

## Подготовка: что должно быть установлено

Перед началом практики убедитесь, что на вашем компьютере установлено:

### Обязательное ПО

- **Python 3.11+** — [Скачать](https://www.python.org/downloads/)
- **Git** — [Скачать](https://git-scm.com/downloads)
- **VS Code** (или другой редактор) — [Скачать](https://code.visualstudio.com/)
- **Аккаунт на GitHub** (или GitLab) — [Зарегистрироваться](https://github.com/signup)

### Проверка установки

Откройте терминал (PowerShell на Windows, Terminal на macOS/Linux) и выполните:

```bash
python --version      # Должно показать Python 3.11 или выше
git --version         # Должно показать версию Git
code --version        # Должно показать версию VS Code
```

---

## Пошаговая инструкция

### Этап 1: Создание репозитория и базовой структуры (60 минут)

#### Шаг 1.1: Создание репозитория (10 мин)

**Что сделать:**

1. Зайти на [GitHub](https://github.com) (или GitLab)
2. Нажать **"New repository"**
3. Заполнить:
   - **Repository name:** `app-conf-project-[название-команды]` (например, `app-conf-project-task-tracker`)
   - **Description:** Краткое описание проекта
   - **Public** (публичный репозиторий)
   - **Initialize with:** НЕ ставить галочки (README, .gitignore создадим сами)
4. Нажать **"Create repository"**

**Полезные ссылки:**

- [Creating a new repository (GitHub Docs)](https://docs.github.com/en/repositories/creating-and-managing-repositories/creating-a-new-repository)

#### Шаг 1.2: Добавление преподавателя как контрибьютора (5 мин)

**Что сделать:**

1. В репозитории перейти в **Settings** → **Collaborators** (или **Manage Access**)
2. Нажать **"Add people"**
3. Ввести: **`andrey-limasov`**
4. Выбрать роль **"Write"** (или **"Developer"** на GitLab)
5. Отправить приглашение

**Важно:** Преподаватель должен быть добавлен **обязательно**, иначе вы не сможете получить оценку!

**Полезные ссылки:**

- [Inviting collaborators (GitHub Docs)](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/managing-repository-settings/managing-access-to-your-repository)

#### Шаг 1.3: Клонирование репозитория (5 мин)

**Что сделать:**

1. В репозитории нажать **"Code"** → скопировать URL (HTTPS или SSH)
2. Открыть терминал в папке, где будете хранить проект
3. Выполнить:

```bash
git clone [URL-репозитория]
cd app-conf-project-[название-команды]
```

**Полезные ссылки:**

- [Cloning a repository (GitHub Docs)](https://docs.github.com/en/repositories/creating-and-managing-repositories/cloning-a-repository)

#### Шаг 1.4: Создание виртуального окружения (10 мин)

**Что сделать:**

**На Windows:**

```bash
python -m venv venv
venv\Scripts\activate
```

**На macOS/Linux:**

```bash
python3 -m venv venv
source venv/bin/activate
```

После активации в терминале появится `(venv)` в начале строки.

**Полезные ссылки:**

- [Virtual Environments (Python Docs)](https://docs.python.org/3/tutorial/venv.html)

#### Шаг 1.5: Создание `.gitignore` (5 мин)

**Что сделать:**
Создать файл `.gitignore` в корне репозитория со следующим содержимым:

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

**Альтернатива:** Использовать [Gitignore.io](https://www.toptal.com/developers/gitignore) — выбрать Python, VisualStudioCode, VirtualEnv, и скопировать результат.

**Полезные ссылки:**

- [Ignoring files (GitHub Docs)](https://docs.github.com/en/get-started/getting-started-with-git/ignoring-files)

#### Шаг 1.6: Создание `README.md` (10 мин)

**Что сделать:**
Создать файл `README.md` со следующим шаблоном (заполнить своими данными):

```markdown
# [Название проекта]

Краткое описание проекта (2-3 предложения).

## Команда

- [Имя 1] — [роль, например: Backend Developer]
- [Имя 2] — [роль, например: DevOps]
- [Имя 3] — [роль, например: Tech Lead]

## Стек технологий

- Python 3.11
- FastAPI
- PostgreSQL
- Docker
- GitHub Actions

## Статус

Проект в разработке.

## Установка и запуск

(Будет добавлено позже)

## Лицензия

MIT
```

**Полезные ссылки:**

- [Awesome README](https://github.com/matiassingers/awesome-readme) — примеры хороших README

#### Шаг 1.7: Создание `CONTRIBUTING.md` (15 мин)

**Что сделать:**
Создать файл `CONTRIBUTING.md` со следующим шаблоном (обсудить и заполнить вместе):

```markdown
# Правила работы в команде

## Git-стратегия

Мы используем [GitFlow / GitHub Flow / Trunk-Based].

### Ветвление
- `main` — стабильная версия
- `develop` — интеграционная ветка (если используем GitFlow)
- `feature/[название]` — новые функции
- `bugfix/[название]` — исправление багов

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

- Частота: [например, 2 раза в неделю]
- Канал связи: [Telegram/Discord/Slack — указать ссылку]
- Время ответа на сообщения: в течение 24 часов

## Разрешение конфликтов

1. Обсуждение в команде
2. Голосование (большинство голосов)
3. Если не можем решить — обращаемся к преподавателю

## Контакты

- Тимлид: [Имя, Telegram]
- Tech Lead: [Имя, Telegram]
```

**Полезные ссылки:**

- [Conventional Commits](https://www.conventionalcommits.org/) — стандарт написания коммитов
- [GitFlow vs GitHub Flow vs Trunk-Based](https://www.flagship.io/git-branching-strategies/) — сравнение стратегий

#### Шаг 1.8: Первый коммит (10 мин)

**Что сделать:**

```bash
git add .gitignore README.md CONTRIBUTING.md
git commit -m "docs: initial project structure and contributing guidelines"
git push origin main
```

**Важно:** Сообщение коммита должно следовать Conventional Commits!

**Полезные ссылки:**

- [Conventional Commits](https://www.conventionalcommits.org/)

---

### Этап 2: Базовый FastAPI и ветвление (70 минут)

#### Шаг 2.1: Установка зависимостей (10 мин)

**Что сделать:**

```bash
pip install fastapi uvicorn[standard]
pip freeze > requirements.txt
```

**Альтернатива (рекомендуется):** Использовать Poetry или uv:

```bash
# Установка Poetry
pip install poetry

# Инициализация проекта
poetry init

# Добавление зависимостей
poetry add fastapi uvicorn[standard]
```

**Полезные ссылки:**

- [FastAPI Installation](https://fastapi.tiangolo.com/tutorial/)
- [Poetry Documentation](https://python-poetry.org/docs/)

#### Шаг 2.2: Создание минимального FastAPI-приложения (15 мин)

**Что сделать:**
Создать файл `main.py` со следующим содержимым:

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

**Полезные ссылки:**

- [FastAPI First Steps](https://fastapi.tiangolo.com/tutorial/first-steps/)
- [FastAPI Bigger Applications](https://fastapi.tiangolo.com/tutorial/bigger-applications/)

#### Шаг 2.3: Создание feature-ветки (5 мин)

**Что сделать:**

```bash
git checkout -b feature/initial-setup
```

**Полезные ссылки:**

- [Git Branching](https://git-scm.com/book/en/v2/Git-Branching-Branches-in-a-Nutshell)

#### Шаг 2.4: Коммит кода (10 мин)

**Что сделать:**

```bash
git add main.py requirements.txt
git commit -m "feat: add initial FastAPI application with health check"
git push origin feature/initial-setup
```

#### Шаг 2.5: Создание Pull Request (10 мин)

**Что сделать:**

1. Зайти на GitHub в репозиторий
2. Увидите предложение "Compare & pull request"
3. Нажать **"Create pull request"**
4. Заполнить:
   - **Title:** `feat: add initial FastAPI application`
   - **Description:** Описать, что добавлено
5. Назначить ревьюера (один из сокомандников)
6. Нажать **"Create pull request"**

**Важно:** Не mergim сразу! Сначала ревью от сокомандника.

**Полезные ссылки:**

- [Creating a pull request (GitHub Docs)](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/creating-a-pull-request)

#### Шаг 2.6: Code Review и merge (15 мин)

**Что сделать:**

1. Ревьюер открывает PR, смотрит код
2. Оставляет комментарии (если есть замечания)
3. Если всё хорошо — нажимает **"Approve"**
4. Автор merge'ит PR (кнопка **"Merge pull request"**)

**Полезные ссылки:**

- [About pull request reviews (GitHub Docs)](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/reviewing-changes-in-pull-requests/about-pull-request-reviews)

#### Шаг 2.7: Проверка работы приложения (5 мин)

**Что сделать:**

```bash
uvicorn main:app --reload
```

Открыть в браузере:

- `http://localhost:8000` — должно показать `{"message": "Hello World"}`
- `http://localhost:8000/health` — должно показать `{"status": "healthy"}`
- `http://localhost:8000/docs` — автоматическая OpenAPI документация

---

### Этап 3: Самопроверка по чек-листу (15 минут)

Каждая команда проходит по чек-листу ниже и убеждается, что всё готово.

---

### Этап 4: Финальная фиксация (10 минут)

Преподаватель обходит команды и фиксирует:

- Название команды
- Ссылку на репозиторий
- Статус выполнения чек-листа

---

## Сводка полезных ссылок

### Git и GitHub:

- [GitHub Docs](https://docs.github.com/)
- [Git Handbook](https://guides.github.com/introduction/git-handbook/)
- [Conventional Commits](https://www.conventionalcommits.org/)
- [Gitignore.io](https://www.toptal.com/developers/gitignore)

### Python и FastAPI:

- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [FastAPI First Steps](https://fastapi.tiangolo.com/tutorial/first-steps/)
- [Python Virtual Environments](https://docs.python.org/3/tutorial/venv.html)
- [Poetry Documentation](https://python-poetry.org/docs/)

### Best Practices:

- [Awesome README](https://github.com/matiassingers/awesome-readme)
- [Git Branching Strategies](https://www.flagship.io/git-branching-strategies/)

---

## Чек-лист самопроверки для команды

Перед сдачей работы убедитесь, что:

### Репозиторий

- [ ] Репозиторий создан и публично доступен
- [ ] Преподаватель (`andrey-limasov`) добавлен как контрибьютор
- [ ] Репозиторий клонирован локально

### Структура проекта

- [ ] Есть `.gitignore` с правильными правилами
- [ ] Есть `README.md` с описанием команды и стека
- [ ] Есть `CONTRIBUTING.md` с правилами работы
- [ ] Есть виртуальное окружение (`venv/`)

### Код

- [ ] Есть `main.py` с FastAPI приложением
- [ ] Есть эндпоинт `GET /` (возвращает `{"message": "Hello World"}`)
- [ ] Есть эндпоинт `GET /health` (возвращает `{"status": "healthy"}`)
- [ ] Есть `requirements.txt` (или `pyproject.toml` для Poetry)

### Git

- [ ] Есть хотя бы 1 коммит в `main`
- [ ] Создана feature-ветка
- [ ] Создан Pull Request
- [ ] PR прошел code review (минимум 1 approval)
- [ ] PR смержен в `main`

### Работа приложения

- [ ] `uvicorn main:app --reload` запускается без ошибок
- [ ] `http://localhost:8000` работает
- [ ] `http://localhost:8000/health` работает
- [ ] `http://localhost:8000/docs` показывает OpenAPI документацию

---

## Дополнительные материалы

### Для тех, кто хочет углубиться

- [Pro Git Book](https://git-scm.com/book/en/v2) — бесплатная книга о Git
- [FastAPI Full Tutorial](https://fastapi.tiangolo.com/tutorial/) — полное руководство по FastAPI
- [Real Python](https://realpython.com/) — качественные статьи по Python
