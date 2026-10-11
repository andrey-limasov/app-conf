# Материалы для самостоятельного изучения (Воркшоп №2)

## 1. Обязательная база

Эти материалы должны прочитать **все** участники курса:

1. **Pro Git (2-е издание)** – Главы 1-3  
   🔗 [Читать онлайн (рус.)](https://git-scm.com/book/ru/v2/)  
   *Очень понятная книга на официальном сайте git.*
2. **Martin Fowler: Branching Patterns**  
   🔗 [Читать статью (англ.)](https://martinfowler.com/articles/branching-patterns.html)  
   *Классическая статья от одного из отцов DevOps. Описывает Feature Branching, Release Branching и их эволюцию. (Для слабых в английском можно использовать встроенный переводчик браузера, суть схем от этого не теряется).*

---

## 2. Бесплатные курсы

**🇷🇺 На русском языке (Stepik):**

- **«Основы Git и GitHub»**  
  🔗 [Курс на stepik.org](https://stepik.org/course/252829/)  
  *Курс бесплатный, оценки отличные*.

**🇬🇧 На английском языке (Coursera):**

- **Introduction to Git and GitHub (by Google)**  
  🔗 [Ссылка на курс](https://www.coursera.org/learn/introduction-git-github)  
  *Один из самых высокооцененных курсов по теме. Его можно пройти **бесплатно в режиме аудита** (Audit). Курс даёт отличное системное понимание того, зачем нужен version control и как работают ветки в реальных командных процессах.*

---

## Видео-курсы на YouTube

**🇷🇺 На русском языке:**

- **GIT, GitHub, GitLab. Полный АКТУАЛЬНЫЙ гайд ЗА ПОЛТОРА ЧАСА**  
  🔗 [Ссылка на видео](https://www.youtube.com/watch?v=0Y-fneoUIO8)  
  *Позволяет за один вечер увидеть весь процесс работы с ветками, merge, rebase и разрешением конфликтов в реальном времени в IDE.*

**🇬🇧 На английском языке:**

- **Git and GitHub for Beginners - Crash Course (by freeCodeCamp.org)**  
  🔗 [Ссылка на видео](https://www.youtube.com/watch?v=RGOj5yH7evk)  
  *freeCodeCamp регулярно обновляет свои гайды. Этот курс считается золотым стандартом англоязычных туториалов: чёткая структура, отличная визуализация веток и реальные примеры команд.*

---

## 4. Интерактивные тренажёры

Чтобы команды могли не только рассказать, но и *показать* работу:

1. **Learn Git Branching**  
   🔗 [https://learngitbranching.js.org/?locale=ru_RU](https://learngitbranching.js.org/?locale=ru_RU)  
   *Хороший визуальный интерактивный тренажёр ветвления.*
2. **GitHub Skills**
  🔗 [https://skills.github.com/](https://skills.github.com/)  
   *Официальные бесплатные интерактивные курсы от GitHub с автоматической проверкой прямо в репозитории.*

---

## Домашнее задание

**Что сделать:** довести черновик ADR 001, начатый на воркшопе, и положить его в репозиторий команды **в отдельную ветку** `docs/adr-001-branching-strategy`. Pull Request **не открывайте**: его откроют на практике №2, и он пройдёт межкомандное ревью. Делает один участник — тот, у кого черновик.

**Дедлайн:** до начала практики №2.

1. Ветка от свежего `main`:

   ```bash
   git checkout main
   git pull origin main
   git checkout -b docs/adr-001-branching-strategy
   mkdir -p docs/adr          # PowerShell: New-Item -ItemType Directory -Force docs/adr
   ```

2. Скопируйте [шаблон ADR](adr-template.md) в `docs/adr/001-branching-strategy.md` (если черновик писали в другом месте — перенесите его по этому пути) и заполните. Статус — «Предложено». Ориентир оформления — [пример](adr-example.md)
3. Коммит и push — команды одинаковы в bash и PowerShell:

   ```bash
   git add docs/adr/001-branching-strategy.md
   git commit -m "docs: add draft ADR 001 on branching strategy"
   git push -u origin HEAD
   ```

   **Ожидаемый вывод** последней команды содержит `* [new branch]      HEAD -> docs/adr-001-branching-strategy` и `branch 'docs/adr-001-branching-strategy' set up to track 'origin/docs/adr-001-branching-strategy'.`

4. GitHub предложит **Compare & pull request** — не нажимайте

**Как проверить:**

```bash
git ls-remote --heads origin docs/adr-001-branching-strategy
```

**Ожидаемый вывод:** строка с SHA и `refs/heads/docs/adr-001-branching-strategy`. На странице репозитория в списке веток есть `docs/adr-001-branching-strategy`, открытых PR с ADR нет.

Если репозиторий команды менялся после практики №1 — пришлите преподавателю новую ссылку: по ним строится кольцо ревью на практике №2.

---

## История изменений

| Дата | Что изменилось |
|---|---|
| 2026-10-11 | Добавлен раздел «Домашнее задание»: черновик ADR 001 — в отдельной ветке `docs/adr-001-branching-strategy` без PR, PR открывается на практике №2 |
