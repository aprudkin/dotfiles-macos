---
name: inbox
description: "Обработка заметок из папки 00 Inbox Obsidian vault"
category: utility
complexity: medium
mcp-servers: []
personas: []
---

# /obs:inbox - Обработка Inbox заметок

Обработка заметок из папки 00 Inbox Obsidian vault.

## Triggers
- Запросы на обработку входящих заметок
- Организация и категоризация необработанного контента
- Разбор папки Inbox по методологии PARA

## Usage
```
/obs:inbox [--all] [--preview] [--batch N]
```

**Аргументы:**
- `--all` — обработать все файлы без подтверждения
- `--preview` — показать что будет обработано без изменений
- `--batch N` — обработать N файлов за раз (по умолчанию: 5)

## Описание

Этот skill анализирует заметки в папке `00 Inbox/` и помогает их организовать:
- Определяет тип контента (cheatsheet, wiki, snippet, bookmark, project)
- Предлагает целевую папку по PARA-методологии
- Добавляет frontmatter с типом и тегами
- Переименовывает файл по конвенции vault

## Behavioral Flow

Когда пользователь вызывает `/obs:inbox`:

1. **Сканирование Inbox**
   ```bash
   ls -la "00 Inbox/"
   ```

2. **Для каждой заметки** в Inbox:
   - Прочитать содержимое файла
   - Определить тип контента по содержимому:
     | Контент | Тип | Целевая папка |
     |---------|-----|---------------|
     | Справочник, команды CLI, how-to | `cheatsheet` | `60 Resources/Cheatsheets/` |
     | Код, функции, примеры | `snippet` | `60 Resources/Code Snippets/` |
     | Ссылки, статьи для чтения | `bookmark` | `60 Resources/Bookmarks/` |
     | Концепция, знания, теория | `wiki` | `30 Wiki/` |
     | Активная задача с дедлайном | `project` | `20 Projects/` |
     | Зона ответственности | `area` | `50 Areas/` |
     | Рабочая задача, тикет JIRA | `work-task` | `50 Areas/Work/Active/` |
     | Медицинский документ | `medical` | `50 Areas/Health/Medical/` |
     | Рецепт на очки | `glasses-prescription` | `50 Areas/Health/Vision/` |

3. **Предложить действия** пользователю через AskUserQuestion:
   - Показать название файла
   - Предложить тип и целевую папку
   - Предложить теги на основе содержимого
   - Предложить новое имя файла (без спецсимволов, читаемое)

4. **После подтверждения**:
   - Добавить YAML frontmatter:
     ```yaml
     ---
     created: YYYY-MM-DD
     updated: YYYY-MM-DD
     type: <тип>
     tags: [<тег1>, <тег2>]
     ---
     ```
   - Создать файл в целевой папке с новым именем
   - Удалить оригинал из Inbox

5. **Повторить** для всех файлов в Inbox

## Пример frontmatter по типам

### cheatsheet
```yaml
---
created: 2026-01-16
updated: 2026-01-16
type: cheatsheet
tags: [cheatsheet, <технология>, <платформа>]
---
```

### wiki
```yaml
---
created: 2026-01-16
updated: 2026-01-16
type: wiki
tags: [wiki, <тема>]
---
```

### snippet
```yaml
---
created: 2026-01-16
updated: 2026-01-16
type: snippet
tags: [snippet, <язык>, <назначение>]
---
```

### bookmark
```yaml
---
created: 2026-01-16
updated: 2026-01-16
type: bookmark
tags: [bookmark, <категория>]
url: <исходный URL если есть>
---
```

### work-task
```yaml
---
created: 2026-01-16
type: work-task
status: active
ticket: JIRA-123
tags: [work]
---
```

### medical
```yaml
---
created: 2026-01-16
type: medical
doctor: Dr. Name
date_of_visit: 2026-01-16
expires: 2027-01-16
tags: [health]
---
```

## Конвенции именования

- Использовать Title Case для названий: `Rclone macOS.md`
- Без спецсимволов кроме пробелов и дефисов
- Краткое описательное название
- Расширение `.md`

## Структура папок vault

```
00 Inbox/              - Необработанные заметки (источник)
10 Daily/              - Дневные заметки
20 Projects/           - Активные проекты
30 Wiki/               - База знаний
40 Templates/          - Шаблоны
50 Areas/              - Зоны ответственности
  ├── Work/Active/     - Текущие рабочие задачи
  ├── Work/Done/       - Завершённые задачи (по кварталам)
  ├── Health/Vision/   - Рецепты на очки
  └── Health/Medical/  - Медицинские документы
60 Resources/          - Справочные материалы
  ├── Bookmarks/
  ├── Cheatsheets/
  └── Code Snippets/
99 System/             - Системные файлы
  └── Attachments/     - Вложения (Images, Documents, Audio)
```

## Обработка ошибок

| Ситуация | Действие |
|----------|----------|
| Папка Inbox пуста | Сообщить пользователю, предложить `/obs:inbox-scan` |
| Файл без расширения .md | Предложить OCR scan или пропустить |
| Duplicate filename | Добавить суффикс `-1`, `-2` к имени |
| Невозможно определить тип | Спросить у пользователя через AskUserQuestion |

## Связанные команды

- `/obs:inbox-scan` — OCR сканирование изображений
- `/obs:session` — сохранение сессии в daily note
- `/obs:learn` — генерация знаний и сохранение в vault

См. также: [[CLAUDE.md]] для полной документации vault
