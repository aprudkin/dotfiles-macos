---
name: rename
description: "Безопасное переименование заметки с обновлением всех ссылок"
category: utility
complexity: medium
mcp-servers: []
personas: []
---

# /obs:rename - Переименование заметки

Безопасное переименование заметки с автоматическим обновлением всех wikilinks в vault.

## Triggers
- Переименование заметки
- Рефакторинг структуры vault
- Исправление названий

## Usage
```
/obs:rename <old-name> <new-name> [--dry-run] [--move <folder>]
```

**Аргументы:**
- `<old-name>` — текущее название заметки (без .md) или путь
- `<new-name>` — новое название
- `--dry-run` — показать что будет изменено без применения
- `--move <folder>` — также переместить в другую папку

## Behavioral Flow

Когда пользователь вызывает `/obs:rename <old> <new>`:

### 1. Поиск исходного файла

```bash
# Найти файл по имени
find . -name "<old>.md" -type f | grep -v ".obsidian"

# Или по точному пути если указан
ls "<old>.md"
```

Если файл не найден — сообщить об ошибке.
Если найдено несколько — показать список и попросить уточнить путь.

### 2. Проверка нового имени

- Файл с новым именем не должен существовать
- Имя не должно содержать запрещённые символы: `/ \ : * ? " < > |`

### 3. Поиск всех ссылок на заметку

```bash
# Найти все файлы со ссылками [[Old Name]] или [[Old Name|alias]]
grep -rn "\[\[<old-name>\]\]" --include="*.md" .
grep -rn "\[\[<old-name>|" --include="*.md" .

# Учесть варианты написания (с путём и без)
# [[Old Name]]
# [[folder/Old Name]]
# [[Old Name|display text]]
```

### 4. Показать план изменений

```markdown
## План переименования

**Исходный файл:** `30 Wiki/Old Article.md`
**Новое имя:** `30 Wiki/New Article.md`

### Файлы со ссылками (5):

| Файл | Строка | Ссылка |
|------|--------|--------|
| `20 Projects/Project A.md` | 15 | `[[Old Article]]` → `[[New Article]]` |
| `20 Projects/Project A.md` | 28 | `[[Old Article|статья]]` → `[[New Article|статья]]` |
| `10 Daily/2026-01-15.md` | 7 | `[[Old Article]]` → `[[New Article]]` |
| `30 Wiki/Index.md` | 42 | `[[Old Article]]` → `[[New Article]]` |
| `30 Wiki/Related.md` | 11 | `[[30 Wiki/Old Article]]` → `[[30 Wiki/New Article]]` |

Применить изменения?
```

### 5. Применение изменений (если не --dry-run)

**Порядок операций:**

a. **Обновить все ссылки** в других файлах:
```bash
# Для каждого файла со ссылкой
# Заменить [[Old Name]] на [[New Name]]
# Заменить [[Old Name|text]] на [[New Name|text]]
# Заменить [[path/Old Name]] на [[path/New Name]]
```

b. **Переименовать файл:**
```bash
mv "<old-path>.md" "<new-path>.md"
```

c. **Обновить внутренние ссылки** в самом файле (если есть self-references):
```bash
# Заменить [[Old Name]] на [[New Name]] внутри файла
```

### 6. Обновить frontmatter (опционально)

Если в frontmatter есть поле `title` — обновить его.

### 7. Git commit

```bash
git add .
git commit -m "Rename: Old Article → New Article

Updated 5 links in 4 files

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>"
git push
```

### 8. Отчёт

```markdown
✅ Переименование завершено

- Файл: `Old Article.md` → `New Article.md`
- Обновлено ссылок: 5
- Затронуто файлов: 4

Коммит: abc1234
```

## Примеры использования

### Простое переименование
```
/obs:rename "Old Article" "New Article"
```

### С полным путём
```
/obs:rename "30 Wiki/Old Article" "30 Wiki/New Article"
```

### Переименование + перемещение
```
/obs:rename "Old Article" "New Article" --move "60 Resources"
```
Переименует и переместит в `60 Resources/New Article.md`.

### Предпросмотр
```
/obs:rename "Old Article" "New Article" --dry-run
```
Покажет план без применения.

## Особенности

### Обработка alias ссылок
```markdown
[[Old Name|display text]]
```
Становится:
```markdown
[[New Name|display text]]
```
Display text сохраняется.

### Обработка путей в ссылках
```markdown
[[folder/Old Name]]
```
Становится:
```markdown
[[folder/New Name]]
```

### Case sensitivity
Поиск ссылок регистронезависимый, но замена сохраняет оригинальный регистр нового имени.

## Обработка ошибок

| Ситуация | Действие |
|----------|----------|
| Файл не найден | Показать похожие имена, предложить выбрать |
| Несколько файлов с таким именем | Показать список, попросить указать путь |
| Новое имя уже существует | Ошибка, предложить другое имя или --force |
| Запрещённые символы | Показать какие символы запрещены |
| Нет ссылок на файл | Просто переименовать файл |

## Ограничения

- Не обновляет ссылки в code blocks (```...```)
- Не обновляет ссылки в комментариях HTML (<!-- -->)
- Не обновляет external markdown ссылки `[text](path.md)`

## Связанные команды

- `/obs:find` — найти заметку для переименования
- `/obs:cleanup` — найти orphans и битые ссылки
- `/obs:retag` — массовая замена тегов
