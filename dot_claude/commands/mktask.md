# /mktask - Go-Task Automation Generator

> Создание задач для go-task (Taskfile.yml) с учётом best practices и особенностей платформы.

## Триггер
```
/mktask <описание автоматизации>
```

## Поведение при вызове

**ВАЖНО: Перед генерацией ВСЕГДА запустить brainstorm для уточнения требований.**

### Шаг 1: Brainstorm (обязательно)
Используй AskUserQuestion для уточнения:

1. **Тип задачи**: sync | docker | build | deploy | backup | utility
2. **Расположение**: глобальный (`~/Taskfile.yml`) или локальный (`./Taskfile.yml`)
3. **Инструменты**: rclone, rsync, docker, npm, etc.
4. **Фильтры**: какие файлы исключить/включить
5. **Безопасность**: нужен ли prompt для подтверждения
6. **Варианты**: нужен ли dry-run, verbose режимы

### Шаг 2: Генерация
После уточнения — сгенерировать YAML.

### Шаг 3: Применение
Спросить куда добавить и применить.

---

## Контекст

**Расположение Taskfile:**
- Глобальный: `~/Taskfile.yml` (вызов через `task -g` или алиас `t`)
- Локальный: `./Taskfile.yml` в директории проекта

**Важные особенности go-task:**

### 1. Переменные директорий
```yaml
vars:
  # Директория откуда вызван task (НЕ где лежит Taskfile!)
  DIR: "{{.USER_WORKING_DIR}}"

  # Директория где лежит Taskfile
  ROOT: "{{.ROOT_DIR}}"

  # Аргументы после --
  ARGS: "{{.CLI_ARGS}}"
```

### 2. Multiline команды
```yaml
cmds:
  - |
    # Bash скрипт внутри
    dirname=$(basename "{{.DIR}}")
    echo "Working in: $dirname"
```

### 3. Флаги задач
```yaml
tasks:
  example:
    desc: "Краткое описание"
    summary: |
      Подробное описание с примерами
      Usage: task example -- arg1 arg2
    interactive: true  # для интерактивных команд
    silent: true       # скрыть вывод самой команды task
    prompt: "Опасная операция. Продолжить?"  # подтверждение
```

### 4. Фильтры для rclone/rsync
```yaml
vars:
  FILTER: >-
    --exclude .DS_Store
    --exclude .git/
    --exclude ".Trash*"
    --exclude Thumbs.db
    --exclude node_modules/
    --exclude __pycache__/
    --exclude "*.pyc"
    --exclude .idea/
    --exclude .vscode/
    --exclude .cache/
    --exclude .claude/
```

### 5. Значения по умолчанию
```yaml
vars:
  HOURS: '{{.CLI_ARGS | default "1"}}'
  ENV: '{{.ENV | default "development"}}'
```

## Поведение

При получении описания автоматизации:

1. **Анализ**: Определить тип задачи (sync, build, deploy, utility)
2. **Локация**: Глобальный или локальный Taskfile
3. **Зависимости**: Проверить наличие утилит (rclone, docker, etc.)
4. **Генерация**: Создать YAML с правильными переменными
5. **Документация**: Добавить desc, summary, примеры использования

## Шаблоны

### Sync задача (rclone)
```yaml
  name:sync:
    desc: "Sync X to Y"
    vars:
      DIR: "{{.USER_WORKING_DIR}}"
      FILTER: >-
        --exclude .DS_Store
        --exclude .git/
    cmds:
      - |
        dirname=$(basename "{{.DIR}}")
        echo "📁 Syncing: {{.DIR}} → remote:/$dirname"
        rclone sync "{{.DIR}}" "remote:/$dirname" {{.FILTER}} --progress
    interactive: true
    silent: true

  name:sync:dry:
    desc: "Dry-run sync"
    vars:
      DIR: "{{.USER_WORKING_DIR}}"
      FILTER: >-
        --exclude .DS_Store
        --exclude .git/
    cmds:
      - |
        dirname=$(basename "{{.DIR}}")
        rclone sync "{{.DIR}}" "remote:/$dirname" {{.FILTER}} --progress --dry-run
    interactive: true
    silent: true
```

### Docker задача
```yaml
  docker:up:
    desc: "Start containers"
    dir: "{{.USER_WORKING_DIR}}"
    cmds:
      - docker compose up -d {{.CLI_ARGS}}

  docker:down:
    desc: "Stop containers"
    dir: "{{.USER_WORKING_DIR}}"
    cmds:
      - docker compose down {{.CLI_ARGS}}

  docker:logs:
    desc: "View logs"
    dir: "{{.USER_WORKING_DIR}}"
    cmds:
      - docker compose logs -f {{.CLI_ARGS}}
    interactive: true
```

### Build задача
```yaml
  build:
    desc: "Build project"
    dir: "{{.USER_WORKING_DIR}}"
    cmds:
      - echo "🔨 Building..."
      - npm run build
    sources:
      - src/**/*
    generates:
      - dist/**/*
```

### Utility задача
```yaml
  util:cleanup:
    desc: "Remove temporary files"
    dir: "{{.USER_WORKING_DIR}}"
    cmds:
      - find . -name "*.pyc" -delete
      - find . -name "__pycache__" -type d -delete
      - find . -name ".DS_Store" -delete
    prompt: "This will delete temp files. Continue?"
```

## Checklist генерации

- [ ] Использовать `{{.USER_WORKING_DIR}}` для текущей директории (не `$PWD`, не `.`)
- [ ] Добавить `desc` для каждой задачи
- [ ] Добавить `interactive: true` для долгих/интерактивных команд
- [ ] Добавить `silent: true` для чистого вывода
- [ ] Добавить `prompt` для опасных операций
- [ ] Создать dry-run версию где уместно
- [ ] Использовать `>-` для multiline переменных (без trailing newline)
- [ ] Использовать `|` для multiline команд
- [ ] Экранировать glob паттерны в кавычках: `--exclude "*.pyc"`

## Примеры вызова

```
/mktask синхронизация ~/Documents с Google Drive
/mktask запуск docker compose с логами
/mktask очистка временных файлов Python
/mktask деплой на сервер через rsync
/mktask бэкап базы данных PostgreSQL
```

## Output

После генерации:
1. Показать готовый YAML
2. Спросить: добавить в глобальный `~/Taskfile.yml` или локальный
3. Обновить справку в Obsidian если нужно
