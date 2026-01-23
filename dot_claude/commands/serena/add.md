---
name: add
description: "Добавление проекта в Serena MCP через Airis Gateway"
category: mcp
complexity: medium
mcp-servers: ["serena", "airis-mcp-gateway-control"]
allowed-tools: "Bash, Read, mcp__airis-mcp-gateway__airis-exec"
personas: []
---

# /serena:add - Добавление проекта в Serena

Регистрация нового проекта в Serena MCP для использования семантического анализа кода через Airis Gateway.

## Triggers
- Нужно добавить новый проект в Serena
- Serena не видит проект (activate_project возвращает ошибку)
- Первичная настройка проекта для работы с Serena

## Usage
```
/serena:add <путь_до_проекта>
```

**Аргументы:**
- `<путь_до_проекта>` — абсолютный путь к директории проекта (обязательный)

**Примеры:**
```
/serena:add /Users/username/projects/my-app
/serena:add ~/OPS/example_bot
```

## Константы

```bash
# Определяется относительно $HOME (не hardcoded)
AIRIS_GATEWAY_DIR="${AIRIS_GATEWAY_DIR:-$HOME/dockers/airis-mcp-gateway}"
SERENA_CONFIG_PATH="/workspaces/serena/config/serena_config.yml"  # Путь внутри контейнера
```

## Требования

- **Airis MCP Gateway** запущен (`docker ps | grep airis-mcp-gateway`)
- **airis-serena** контейнер запущен (`docker ps | grep airis-serena`)
- **SERENA_MODE=serena-remote** в конфигурации Gateway
- Путь к проекту должен быть смонтирован в контейнер airis-serena

## Behavioral Flow

Когда пользователь вызывает `/serena:add <path>`:

### 0. Проверка контейнеров
```bash
# Проверить что Gateway и Serena запущены
docker ps --filter "name=airis-mcp-gateway" --filter "name=airis-serena" --format "table {{.Names}}\t{{.Status}}"
```

**Если airis-serena не запущен:**
```
## Ошибка: контейнер airis-serena не запущен

Serena работает в режиме serena-remote, но контейнер не запущен.

### Запустите контейнер:
```bash
cd ~/dockers/airis-mcp-gateway && docker compose --profile serena-remote up -d serena
```

Или используйте полный перезапуск:
```bash
cd ~/dockers/airis-mcp-gateway && task docker:restart
```
```

### 1. Валидация пути
```bash
# Развернуть ~ в полный путь если нужно
FULL_PATH=$(eval echo "<path>")

# Проверить что директория существует
ls -la "$FULL_PATH"

# Проверить наличие .serena/project.yml (опционально)
ls "$FULL_PATH/.serena/project.yml" 2>/dev/null || echo "No .serena config (will be created on first activation)"
```

### 2. Проверка SERENA_MODE (КРИТИЧНО!)
```bash
# Проверить режим Serena в Gateway контейнере
docker exec airis-mcp-gateway env | grep SERENA_MODE
```

**Если SERENA_MODE != serena-remote:**

⚠️ **ОСТАНОВИТЬ выполнение и сообщить пользователю:**

```
## Ошибка конфигурации

SERENA_MODE установлен в "${current_mode}", требуется "serena-remote".

### Как исправить:

1. Создайте/проверьте файл .env в ~/dockers/airis-mcp-gateway/:
   ```
   SERENA_MODE=serena-remote
   ```

2. Или добавьте в docker-compose.override.yml:
   ```yaml
   services:
     api:
       environment:
         - SERENA_MODE=serena-remote
   ```

3. Перезапустите Gateway:
   ```bash
   cd ~/dockers/airis-mcp-gateway && task docker:restart
   ```
```

**Если SERENA_MODE=serena-remote — продолжить.**

### 3. Проверка монтирования пути
```bash
# Проверить доступность пути внутри airis-serena
docker exec airis-serena ls -la "$FULL_PATH" 2>/dev/null || echo "Path not accessible"
```

**Если путь недоступен:**

```
## Ошибка: путь не смонтирован

Путь "$FULL_PATH" недоступен внутри контейнера airis-serena.

### Как исправить:

Добавьте монтирование в docker-compose.override.yml:

```yaml
services:
  serena:
    volumes:
      - $HOME:$HOME:rw
```

Затем перезапустите:
```bash
cd ~/dockers/airis-mcp-gateway && task docker:restart
```
```

### 4. Добавление проекта в конфигурацию Serena
```bash
# Проверить, не добавлен ли уже проект
docker exec airis-serena grep -q "$FULL_PATH" /workspaces/serena/config/serena_config.yml && echo "Already registered" || echo "Not registered"
```

**Если уже зарегистрирован — перейти к шагу 7 (верификация).**

```bash
# Добавить проект в список
docker exec airis-serena sh -c "sed -i 's|^projects:|projects:\n- $FULL_PATH|' /workspaces/serena/config/serena_config.yml"

# Проверить результат
docker exec airis-serena grep "$FULL_PATH" /workspaces/serena/config/serena_config.yml
```

### 5. Перезапуск Serena для применения изменений
```bash
# Перезапустить контейнер Serena
docker restart airis-serena

# Дождаться готовности (healthcheck с таймаутом)
for i in {1..12}; do
  STATUS=$(docker ps --filter "name=airis-serena" --format "{{.Status}}")
  if [[ "$STATUS" == *"healthy"* ]]; then
    echo "✅ airis-serena is healthy"
    break
  fi
  echo "Waiting for airis-serena to be healthy... ($i/12)"
  sleep 5
done

# Финальная проверка
docker ps --filter "name=airis-serena" --format "{{.Names}}\t{{.Status}}"
```

### 6. Переподключение Gateway к Serena

Использовать MCP для сброса кэша соединения:

```
airis-exec: airis-mcp-gateway-control:gateway_disable_server {"server_name": "serena"}
airis-exec: airis-mcp-gateway-control:gateway_enable_server {"server_name": "serena"}
```

### 7. Верификация

Активировать проект через MCP:
```
airis-exec: serena:activate_project {"project": "<project_name>"}
```

Где `<project_name>`:
- Имя из `.serena/project.yml` (поле `project_name`)
- Или имя директории (basename пути)
- Или полный путь

### 8. Вывод результата

**При успехе:**
```
## Проект добавлен в Serena

✅ Путь: $FULL_PATH
✅ Имя проекта: <name>
✅ Статус: активирован

### Доступные Serena инструменты:
- `serena:find_symbol` — поиск символов в коде
- `serena:get_symbols_overview` — обзор структуры файла
- `serena:think_about_*` — инструменты рефлексии
- `serena:write_memory` — сохранение заметок о проекте
- `serena:list_memories` — список сохранённых заметок
```

**При ошибке:**
```
## Ошибка добавления проекта

❌ Причина: <описание ошибки>

### Диагностика:
<логи и детали>

### Рекомендации:
<шаги для исправления>
```

## Проверки перед выполнением (Checklist)

| # | Проверка | Команда | Ожидаемый результат |
|---|----------|---------|---------------------|
| 1 | Gateway работает | `docker ps \| grep airis-mcp-gateway` | Up (healthy) |
| 2 | Serena контейнер работает | `docker ps \| grep airis-serena` | Up (healthy) |
| 3 | SERENA_MODE=serena-remote | `docker exec airis-mcp-gateway env \| grep SERENA_MODE` | serena-remote |
| 4 | Путь существует | `ls -la <path>` | Директория существует |
| 5 | Путь смонтирован | `docker exec airis-serena ls <path>` | Директория доступна |

## Обработка ошибок

| Ошибка | Причина | Решение |
|--------|---------|---------|
| `SERENA_MODE=serena-local` | Неправильная конфигурация | Добавить `SERENA_MODE=serena-remote` в .env |
| `Path not accessible` | Путь не смонтирован | Добавить volume в docker-compose.override.yml |
| `Already registered` | Проект уже добавлен | Пропустить добавление, перейти к верификации |
| `ProjectNotFoundError` | Кэш Gateway не обновился | Disable/enable serena через gateway control |
| `Container not running` | airis-serena остановлен | `task docker:restart` или запустить с --profile |

## Архитектура (справка)

```
┌─────────────────────────────────────────────────────────────┐
│  HOST (macOS/Linux)                                         │
│  $HOME/projects/...                                         │
│           │                                                 │
│           ▼ (volume mount: $HOME:$HOME)                     │
│  ┌────────────────────────────────────────────────────────┐ │
│  │ airis-serena (Docker)                                  │ │
│  │                                                        │ │
│  │ /workspaces/serena/config/serena_config.yml            │ │
│  │   └── projects:                                        │ │
│  │         - /home/user/projects/app1                     │ │
│  │         - /home/user/projects/app2                     │ │
│  └────────────────────────────────────────────────────────┘ │
│           ▲                                                 │
│           │ (SSE: http://serena:8000/sse)                   │
│           │                                                 │
│  ┌────────────────────────────────────────────────────────┐ │
│  │ airis-mcp-gateway (Docker)                             │ │
│  │ SERENA_MODE=serena-remote ← КРИТИЧНО!                  │ │
│  └────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

## Связанные команды

- `/sc:reflect` — рефлексия с использованием Serena
- `/sc:load` — загрузка контекста проекта
- `task docker:restart` — перезапуск Airis Gateway стека
