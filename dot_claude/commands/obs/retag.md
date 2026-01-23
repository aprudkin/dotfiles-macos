---
name: retag
description: "Массовая замена тегов в vault (frontmatter и текст)"
category: utility
complexity: medium
mcp-servers: []
personas: []
---

# /obs:retag - Массовая замена тегов

Замена тега во всём vault: в frontmatter и в тексте заметок.

## Triggers
- Переименование тегов
- Консолидация похожих тегов
- Рефакторинг системы тегов

## Usage
```
/obs:retag <old-tag> <new-tag> [--dry-run] [--folder <folder>] [--delete]
```

**Аргументы:**
- `<old-tag>` — текущий тег (без #)
- `<new-tag>` — новый тег (без #)
- `--dry-run` — показать что будет изменено
- `--folder <folder>` — применить только к папке
- `--delete` — удалить тег вместо замены (new-tag не нужен)

## Behavioral Flow

Когда пользователь вызывает `/obs:retag <old> <new>`:

### 1. Поиск файлов с тегом

Теги могут быть в двух местах:

**a. В frontmatter:**
```yaml
---
tags: [old-tag, other]
# или
tags:
  - old-tag
  - other
---
```

**b. В тексте:**
```markdown
Текст с #old-tag в середине.
```

```bash
# Поиск в frontmatter (массив)
grep -rn "tags:.*old-tag" --include="*.md" .

# Поиск в тексте
grep -rn "#old-tag\b" --include="*.md" .
```

### 2. Показать план изменений

```markdown
## План замены тега

**Старый тег:** `#old-tag`
**Новый тег:** `#new-tag`

### Файлы с тегом (12):

| Файл | Место | Изменение |
|------|-------|-----------|
| `20 Projects/Alpha.md` | frontmatter | `tags: [old-tag, project]` → `tags: [new-tag, project]` |
| `20 Projects/Beta.md` | frontmatter | `tags: [old-tag]` → `tags: [new-tag]` |
| `30 Wiki/Article.md` | текст:15 | `#old-tag` → `#new-tag` |
| `30 Wiki/Guide.md` | frontmatter + текст | 2 замены |
| ... | | |

**Всего:** 12 файлов, 15 замен

Применить?
```

### 3. Применение изменений

**a. Замена в frontmatter:**

Формат массива в одну строку:
```yaml
# До
tags: [old-tag, other, tags]
# После
tags: [new-tag, other, tags]
```

Формат списка:
```yaml
# До
tags:
  - old-tag
  - other
# После
tags:
  - new-tag
  - other
```

**b. Замена в тексте:**
```markdown
# До
Текст с #old-tag здесь.
# После
Текст с #new-tag здесь.
```

**Важно:** заменять только целые теги, не части слов:
- `#old-tag` → заменить
- `#old-tag-extended` → НЕ заменять
- `old-tag` (без #) → НЕ заменять

### 4. Git commit

```bash
git add .
git commit -m "Retag: #old-tag → #new-tag

Updated 15 occurrences in 12 files

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>"
git push
```

### 5. Отчёт

```markdown
✅ Замена тегов завершена

- Тег: `#old-tag` → `#new-tag`
- Файлов обновлено: 12
- Всего замен: 15
  - В frontmatter: 10
  - В тексте: 5

Коммит: def5678
```

## Примеры использования

### Простая замена
```
/obs:retag project proj
```
Заменит `#project` на `#proj` везде.

### Предпросмотр
```
/obs:retag old-tag new-tag --dry-run
```

### Только в папке
```
/obs:retag work work-task --folder "50 Areas/Work"
```

### Удаление тега
```
/obs:retag obsolete-tag --delete
```
Удалит `#obsolete-tag` из всех файлов.

### Слияние тегов
```
/obs:retag javascript js
/obs:retag JS js
/obs:retag JavaScript js
```
Консолидация разных написаний в один тег.

## Особенности

### Nested tags
Obsidian поддерживает вложенные теги: `#parent/child`

```
/obs:retag parent/old parent/new
```
Заменит `#parent/old` на `#parent/new`.

### Case sensitivity
- Поиск case-insensitive (найдёт #Tag и #tag)
- Замена использует точный регистр нового тега

### Множественные теги в frontmatter
```yaml
tags: [tag1, old-tag, tag2, old-tag]
```
Заменит все вхождения.

## Обработка ошибок

| Ситуация | Действие |
|----------|----------|
| Тег не найден | Сообщить, предложить похожие теги |
| Новый тег уже существует | Предупредить о слиянии, спросить подтверждение |
| Некорректный формат тега | Показать правила именования |
| Тег в code block | Пропустить (не заменять в ```) |

## Правила именования тегов

- Только буквы, цифры, дефис, подчёркивание, слэш
- Без пробелов
- Не начинается с цифры
- Рекомендуется kebab-case: `my-tag`

## Связанные команды

- `/obs:find --tag` — найти файлы с тегом
- `/obs:cleanup` — найти проблемы в vault
- `/obs:rename` — переименование заметок
