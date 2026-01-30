# Global Claude Code Instructions

## Date & Time

**ВАЖНО: Никогда не полагаться на env или вычисления в уме для даты/времени.**

Перед любой операцией, связанной с датой или временем:
```bash
date "+%Y-%m-%d %H:%M %A"  # 2026-01-26 13:30 Monday
```

Использовать результат `date` для:
- Дня недели в заголовках
- Времени в таймлайнах
- Определения типа рефлексии (morning/evening/etc)
- Любых frontmatter с датами

## macOS

**Версия:** macOS Sequoia (15.x)

При ответах на вопросы о macOS:
- Учитывать изменения UI в Sequoia (новые пути в System Settings)
- Extensions теперь в **System Settings → General → Login Items & Extensions**
- Не использовать устаревшие пути из Monterey/Ventura

## Taskfile Maintenance

**ВАЖНО: При изменении `~/Taskfile.yml` всегда обновлять справку в Obsidian.**

После добавления/изменения/удаления задач в `~/Taskfile.yml`:
1. Обновить `~/Obsidian/obsidian/00 Inbox/Go-Task Global Commands.md`
2. Добавить новую команду в таблицу
3. Обновить дату в footer (`*Обновлено: YYYY-MM-DD*`)

**Справка должна содержать:**
- Название команды
- Краткое описание
- Примеры использования (если нетривиально)

## Zsh Aliases Maintenance

**ВАЖНО: При добавлении/изменении алиасов в `~/.zshrc` всегда обновлять справку в Obsidian.**

После изменения алиасов:
1. Обновить `~/Obsidian/obsidian/60 Resources/Cheatsheets/Zsh Aliases.md`
2. Добавить новый алиас в соответствующую таблицу
3. Обновить `updated:` в frontmatter
4. Синхронизировать dotfiles: `chezmoi re-add`

## Skills Maintenance

При создании нового skill в `~/.claude/commands/`:
1. Документировать в Obsidian если skill сложный
2. Обновить `updated:` в соответствующих справках

## Obsidian Vault

**Путь:** `~/Obsidian/obsidian/`

**Inbox:** `00 Inbox/` — для новых справок и заметок

**Git:** После создания/изменения файлов в vault — коммитить изменения
