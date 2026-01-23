---
name: cleanup
description: "Диагностика vault: orphans, битые ссылки, пустые файлы, проблемы frontmatter"
category: utility
complexity: medium
mcp-servers: []
personas: []
---

# /obs:cleanup - Диагностика и очистка Vault

Анализ vault на проблемы: orphan notes, битые ссылки, пустые файлы, отсутствующий frontmatter.

## Triggers
- Периодическое обслуживание vault
- Поиск проблемных заметок
- Подготовка к рефакторингу

## Usage
```
/obs:cleanup [--fix] [--type <type>] [--dry-run]
```

**Аргументы:**
- `--fix` — автоматически исправить найденные проблемы
- `--type <type>` — проверить только указанный тип (orphans, broken, empty, frontmatter)
- `--dry-run` — показать что будет исправлено без изменений

## Behavioral Flow

Когда пользователь вызывает `/obs:cleanup`:

### 1. Сканирование vault

```bash
# Получить список всех markdown файлов
find . -name "*.md" -type f | grep -v ".obsidian" | grep -v "node_modules"
```

### 2. Проверка Orphan Notes

Orphan — заметка без входящих ссылок (никто на неё не ссылается).

**Исключения (не считаются orphans):**
- Файлы в `00 Inbox/`
- Файлы в `10 Daily/`
- Файлы в `40 Templates/`
- Файлы в `99 System/`
- `CLAUDE.md`, `GEMINI.md`, `README.md`

**Алгоритм:**
```bash
# 1. Собрать все wikilinks из всех файлов
grep -roh "\[\[[^]]*\]\]" --include="*.md" . | sort | uniq

# 2. Для каждого файла проверить есть ли на него ссылка
# Если нет — это orphan
```

### 3. Проверка Broken Links

Broken link — `[[Note Name]]` где файл `Note Name.md` не существует.

```bash
# 1. Извлечь все wikilinks
grep -roh "\[\[[^]|]*" --include="*.md" . | sed 's/\[\[//' | sort | uniq

# 2. Для каждой ссылки проверить существование файла
# Учитывать что ссылка может быть с путём или без
```

### 4. Проверка Empty Files

```bash
# Файлы размером 0 байт или только с frontmatter
find . -name "*.md" -size 0
```

Также проверить файлы где после frontmatter нет контента.

### 5. Проверка Frontmatter

Проверить что каждый файл (кроме Templates) имеет:
- Валидный YAML frontmatter (`---` ... `---`)
- Поле `type`
- Поле `created`

### 6. Генерация отчёта

```markdown
# Vault Cleanup Report

## Summary
| Проблема | Количество |
|----------|------------|
| Orphan notes | 5 |
| Broken links | 3 |
| Empty files | 2 |
| Missing frontmatter | 4 |

## Orphan Notes (5)
Заметки без входящих ссылок:

1. `30 Wiki/Old Article.md`
   - Создана: 2025-06-15
   - Действие: [Удалить] [Добавить ссылку] [Пропустить]

2. `60 Resources/Snippets/unused.md`
   ...

## Broken Links (3)
Ссылки на несуществующие файлы:

1. `[[Missing Note]]` в `20 Projects/Project A.md:15`
   - Действие: [Создать заметку] [Удалить ссылку] [Пропустить]

## Empty Files (2)
...

## Missing Frontmatter (4)
...
```

### 7. Интерактивное исправление (если --fix)

Для каждой проблемы предложить действие через AskUserQuestion:

**Orphans:**
- Удалить файл
- Оставить (добавить в исключения)
- Создать ссылку из Index/MOC

**Broken Links:**
- Создать заметку с таким именем
- Удалить ссылку
- Заменить на другую ссылку

**Empty Files:**
- Удалить
- Оставить

**Missing Frontmatter:**
- Добавить базовый frontmatter
- Пропустить

### 8. Git commit (если были изменения)

```bash
git add .
git commit -m "Vault cleanup: fix N issues

- Deleted X orphan notes
- Fixed Y broken links
- Removed Z empty files
- Added frontmatter to W files

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>"
git push
```

## Примеры использования

### Полная диагностика
```
/obs:cleanup
```
Показывает отчёт по всем проблемам.

### Только orphans
```
/obs:cleanup --type orphans
```

### Автоматическое исправление
```
/obs:cleanup --fix
```
Интерактивно предлагает действия для каждой проблемы.

### Предпросмотр исправлений
```
/obs:cleanup --fix --dry-run
```

## Обработка ошибок

| Ситуация | Действие |
|----------|----------|
| Vault пустой | Показать сообщение, выйти |
| Нет проблем | Показать "Vault в отличном состоянии!" |
| Git не инициализирован | Пропустить commit, предупредить |

## Связанные команды

- `/obs:rename` — переименование с обновлением ссылок
- `/obs:retag` — массовая замена тегов
- `/obs:find` — поиск по vault
