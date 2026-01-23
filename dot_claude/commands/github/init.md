---
name: init
description: "Инициализация Git репозитория и создание GitHub repo с автоматическим push"
category: devops
complexity: low
mcp-servers: []
personas: []
---

# /github:init - Инициализация GitHub репозитория

Создание GitHub репозитория, инициализация локального git и push.

## Triggers
- Новый проект без git
- Нужно быстро создать repo на GitHub
- Инициализация и первый push

## Usage
```
/github:init [путь_к_папке] [--public] [--name <repo-name>]
```

**Аргументы:**
- `[путь_к_папке]` — путь к папке проекта (по умолчанию: текущая директория)
- `--public` — создать публичный репозиторий (по умолчанию: приватный)
- `--name <repo-name>` — имя репозитория (по умолчанию: имя папки)

## Требования

- **gh** — GitHub CLI с настроенной авторизацией
- **git** — установленный git

## Behavioral Flow

Когда пользователь вызывает `/github:init [path]`:

1. **Определение директории**
   ```bash
   # Если путь указан — использовать его
   # Если не указан — использовать текущую директорию
   cd "<target_dir>"
   ```

2. **Проверка окружения**
   ```bash
   # Проверить авторизацию GitHub CLI
   gh auth status

   # Проверить что директория существует
   ls -la "<target_dir>"
   ```

3. **Проверка git статуса**
   ```bash
   # Проверить инициализирован ли git
   git rev-parse --git-dir 2>/dev/null
   ```

4. **Если git НЕ инициализирован:**
   ```bash
   # Инициализировать git
   git init

   # Создать .gitignore если нет
   # (базовый для macOS/IDE)

   # Добавить все файлы
   git add .

   # Первый коммит
   git commit -m "Initial commit"
   ```

5. **Проверка remote**
   ```bash
   # Проверить есть ли уже remote origin
   git remote -v
   ```

6. **Если remote НЕ настроен:**
   ```bash
   # Проверить название текущей ветки
   BRANCH=$(git branch --show-current)

   # Создать репозиторий на GitHub и запушить
   # --private: приватный репо (по умолчанию)
   # --source=.: использовать текущую папку
   # --remote=origin: добавить как origin
   # --push: сразу запушить
   gh repo create <repo-name> --private --source=. --remote=origin --push
   ```

7. **Если remote УЖЕ настроен:**
   ```bash
   # Определить текущую ветку
   BRANCH=$(git branch --show-current)

   # Запушить (main или master в зависимости от ветки)
   git push -u origin "$BRANCH"
   ```

8. **Вывод результата**
   ```
   ✅ Репозиторий создан: https://github.com/<user>/<repo>
   ✅ Запушено в origin/main
   ```

## Примеры использования

### Инициализация текущей папки
```
/github:init
```

### Инициализация указанной папки
```
/github:init /Users/username/projects/my-app
```

### Создать публичный репозиторий
```
/github:init --public
```

### Указать имя репозитория
```
/github:init --name awesome-project
```

### Полный пример
```
/github:init /path/to/project --public --name my-awesome-app
```

## Базовый .gitignore (создаётся автоматически если нет)

```gitignore
# macOS
.DS_Store
._*

# IDE
.idea/
.vscode/
*.swp
*.swo

# Dependencies
node_modules/
vendor/
__pycache__/
*.pyc

# Environment
.env
.env.local
*.local

# Build
dist/
build/
*.log
```

## Обработка ошибок

| Ошибка | Причина | Решение |
|--------|---------|---------|
| `gh: command not found` | gh не установлен | `brew install gh` |
| `not logged in` | Нет авторизации | `gh auth login` |
| `repository already exists` | Репо уже есть на GitHub | Использовать `git remote add` вручную |
| `remote origin already exists` | Remote уже настроен | Пропустить создание, только push |

## Связанные команды

- `/gitlab:clone` — клонирование GitLab репозиториев
- `/gitlab:audit` — аудит GitLab репозиториев
