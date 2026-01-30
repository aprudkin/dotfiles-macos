# /obs:renew:articles - Синхронизация документации с системой

Автоматическая актуализация статей в Obsidian vault на основе реального состояния macOS.

## Triggers
- Запросы на обновление документации настроек
- Синхронизация статей с конфигами системы
- Проверка актуальности документации

## Usage
```
/obs:renew:articles [--source <name>] [--preview] [--create-missing]
```

**Аргументы:**
- `--source <name>` — обновить только указанный источник (go-task, homebrew, chezmoi, zsh, ssh, git, age, skills)
- `--preview` — показать diff без изменений
- `--create-missing` — создать недостающие статьи

## Источники данных

| Источник | Команда сбора | Setup статья | Справочник |
|----------|---------------|--------------|------------|
| **go-task** | `task --global --list-all` | `50 Areas/Setup/Go-Task.md` | `Go-Task Cheatsheet.md` |
| **homebrew** | `brew leaves` | `50 Areas/Setup/Homebrew.md` | `Homebrew Cheatsheet.md` |
| **chezmoi** | `chezmoi managed` | `50 Areas/Setup/Chezmoi.md` | `Chezmoi Cheatsheet.md` |
| **zsh** | `grep "^alias" ~/.zshrc` | `50 Areas/Setup/Zsh Aliases.md` | `60 Resources/Cheatsheets/Zsh Aliases.md` |
| **ssh** | `grep "^Host" ~/.ssh/config` | `50 Areas/Setup/SSH Hosts.md` | `SSH Cheatsheet.md` |
| **git** | `git config --global --list \| grep alias` | `50 Areas/Setup/Git Aliases.md` | `Git Cheatsheet.md` |
| **age** | `~/.config/chezmoi/chezmoi.toml` | `50 Areas/Setup/Age Keys.md` | `Age Encryption.md`, `SOPS и Age.md` |
| **skills** | `chezmoi managed \| grep .claude/commands` | `50 Areas/Setup/Claude Skills.md` | `Claude Code Cheatsheet.md` |

## Два уровня документации

### Setup (личное, авто-обновляемое)
- Расположение: `50 Areas/Setup/`
- Содержимое: конкретные настройки пользователя
- Обновляется: автоматически через этот skill
- Пример: список моих 42 Homebrew пакетов

### Cheatsheets (общее, ручное)
- Расположение: `60 Resources/Cheatsheets/`
- Содержимое: как использовать инструмент
- Обновляется: вручную
- Пример: синтаксис команд `brew install/upgrade/...`

### Связи
Каждая Setup статья содержит секцию "См. также" с ссылками на:
1. Другие связанные Setup статьи
2. Соответствующие Cheatsheets

## Behavioral Flow

### 1. Сбор данных
Для каждого источника выполнить соответствующую команду.

### 2. Парсинг и форматирование

#### go-task
```bash
task --global --list-all
```
Формат вывода:
```
* command:    Description      (aliases: alias1, alias2)
```
Парсить в таблицу:
| Команда | Описание | Алиасы |
|---------|----------|--------|

#### homebrew
```bash
brew leaves
```
Список пакетов по одному на строку. Группировать по категориям:
- Development: go, node, python
- CLI tools: fzf, ripgrep, eza
- Security: age, sops, pass
- Media: yt-dlp, whisper-cpp, imagemagick
- Network: rclone, cloudflared, openconnect

#### chezmoi
```bash
chezmoi managed
```
Группировать по папкам:
- `.claude/` — Claude Code конфиги
- `.config/` — конфиги приложений
- `Library/Services/` — Quick Actions
- Dotfiles — `.zshrc`, `.gitconfig` и т.д.

#### zsh
```bash
grep -E "^alias " ~/.zshrc
```
Парсить формат `alias name="command"` в таблицу.

#### ssh
```bash
grep -E "^Host " ~/.ssh/config
```
Показать список хостов (без деталей подключения из соображений безопасности).

#### git
```bash
git config --global --list | grep "^alias"
```
Парсить формат `alias.name=command` в таблицу.

#### age
```bash
cat ~/.config/chezmoi/chezmoi.toml
```
Извлечь:
- encryption method
- recipient (public key)
- identity path

**ВАЖНО:** Никогда не показывать приватные ключи! Только public keys и пути.

#### skills
```bash
chezmoi managed | grep ".claude/commands"
```
Парсить структуру:
- Namespace (obs, gitlab, sc, etc.)
- Skill name
- Description из frontmatter

### 3. Сравнение с текущей статьёй

Читать существующую статью и найти секцию между маркерами:
```markdown
<!-- BEGIN:AUTO:<SOURCE> -->
...content...
<!-- END:AUTO:<SOURCE> -->
```

Если секция отличается — обновить.

### 4. Обновление статьи

1. Заменить содержимое между маркерами
2. Обновить `updated:` в frontmatter на текущую дату
3. Сохранить файл

### 5. Commit

```bash
git add "50 Areas/Setup/*.md"
git commit -m "docs: renew articles from system config"
```

## Формат статей

### Frontmatter
```yaml
---
created: 2026-01-29
updated: 2026-01-29
type: setup
tags: [setup, <source-specific-tags>]
auto-renew: true
source: <source-name>
---
```

### Структура статьи
```markdown
# Title

Краткое описание.

## Auto-generated section

<!-- BEGIN:AUTO:<SOURCE> -->
| Команда | Описание |
...
<!-- END:AUTO:<SOURCE> -->

## Manual notes (optional)

Ручные заметки, которые не перезаписываются.
```

## Шаблоны статей

### Go-Task.md
```markdown
---
created: 2026-01-29
updated: 2026-01-29
type: setup
tags: [setup, go-task, automation]
auto-renew: true
source: go-task
---

# Go-Task Commands

Глобальные команды из `~/Taskfile.yml`.

## Вызов

\`\`\`bash
t <command>           # через алиас
task -g <command>     # явно глобальный
\`\`\`

## Команды

<!-- BEGIN:AUTO:GO-TASK -->
| Команда | Описание |
|---------|----------|
<!-- END:AUTO:GO-TASK -->

## См. также

- [[Chezmoi]]
- [[Zsh Aliases]]
```

### Homebrew.md
```markdown
---
created: 2026-01-29
updated: 2026-01-29
type: setup
tags: [setup, homebrew, macos]
auto-renew: true
source: homebrew
---

# Homebrew Packages

Установленные пакеты (top-level, без зависимостей).

## Установка

\`\`\`bash
brew install <package>
brew uninstall <package>
brew upgrade
\`\`\`

## Пакеты

<!-- BEGIN:AUTO:HOMEBREW -->
| Категория | Пакеты |
|-----------|--------|
<!-- END:AUTO:HOMEBREW -->
```

### Age Keys.md
```markdown
---
created: 2026-01-29
updated: 2026-01-29
type: setup
tags: [setup, age, sops, encryption, security]
auto-renew: true
source: age
---

# Age Keys

Конфигурация age/SOPS шифрования.

## Ключи

<!-- BEGIN:AUTO:AGE -->
| Параметр | Значение |
|----------|----------|
| Encryption | age |
| Public Key | `age1...` |
| Identity | `~/.config/chezmoi/key.txt` |
<!-- END:AUTO:AGE -->

## Использование

\`\`\`bash
# SOPS с age
sops secrets.yaml

# Прямое шифрование
age -r age1... -o file.age file.txt
\`\`\`

## Бэкап ключа

Приватный ключ хранится в 1Password:
- Vault: Backup
- Item: "Chezmoi Age Key"

## См. также

- [[SOPS и Age]] — общая инструкция
- [[Chezmoi]]
```

## Обработка ошибок

| Ситуация | Действие |
|----------|----------|
| Команда не найдена | Пропустить источник, предупредить |
| Статья не существует | Создать по шаблону (с `--create-missing`) |
| Нет маркеров в статье | Добавить маркеры в конец |
| Git не инициализирован | Пропустить commit |

## Безопасность

**НИКОГДА не включать:**
- Приватные ключи
- Пароли
- Токены API
- IP адреса внутренних серверов (из SSH config)

**МОЖНО включать:**
- Публичные ключи
- Имена хостов (без IP)
- Команды и алиасы
- Список пакетов

## Связанные статьи

Skill автоматически обновляет:
- `50 Areas/Setup/*.md`
- `60 Resources/Cheatsheets/Zsh Aliases.md` — алиасы из ~/.zshrc

Статьи с общими знаниями НЕ обновляются:
- `60 Resources/Cheatsheets/SOPS и Age.md`
- `60 Resources/Cheatsheets/Homebrew Cheatsheet.md`
