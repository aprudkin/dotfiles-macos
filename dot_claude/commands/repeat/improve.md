---
name: improve
description: "Автоматическое выполнение /sc:improve quality N раз с полным циклом (analyze → fix → test → commit → deploy)"
category: automation
complexity: advanced
---

# /repeat:improve - Многократное улучшение качества кода

Автоматически выполняет `/sc:improve quality` указанное количество раз с полным циклом:
1. Анализ кода
2. Исправление найденных проблем
3. Запуск тестов
4. Коммит изменений
5. Деплой в продакшен

## Использование
```
/repeat:improve <N>
```
Где `<N>` — количество раундов улучшений (по умолчанию 3).

## Условия остановки

Выполнение ОСТАНАВЛИВАЕТСЯ досрочно если:
1. **Зацикливание** — те же проблемы появляются повторно после исправления
2. **Регрессия** — количество новых проблем превышает количество исправленных
3. **Чистый код** — не найдено проблем для исправления
4. **Провал тестов** — тесты не проходят после исправлений

## Полный цикл для каждого раунда

### Фаза 1: Анализ
```bash
uv run python -m flake8 <target>/ --select=F401,F841
uv run mypy <target>/
```
Поиск:
- Неиспользуемых импортов (F401)
- Неиспользуемых переменных (F841)
- Ошибок типизации (mypy)
- Inline-запросов без helper-функций
- Отсутствующих generic type hints (`dict` → `dict[str, Any]`)

### Фаза 2: Исправление
Применить исправления используя Edit tool:
- Удалить неиспользуемые импорты
- Заменить inline-запросы на helper-вызовы
- Добавить generic type hints
- Исправить ошибки mypy

### Фаза 3: Валидация
```bash
task test:local  # или эквивалент для проекта
```
- ВСЕ тесты должны пройти
- Если тесты падают — откатить изменения и остановиться

### Фаза 4: Версионирование
Увеличить PATCH версию в:
- `pyproject.toml`
- `REPO_INDEX.md` (если есть)

### Фаза 5: Коммит и деплой
```bash
git add -A
git commit -m "refactor: code quality improvements round N (vX.Y.Z)"
git push
task deploy  # если есть
gh release create vX.Y.Z --title "vX.Y.Z" --notes "..."
```

## Отслеживание прогресса

Используй TodoWrite для трекинга:
```
Round 1: Complete (v1.2.5) ✓
Round 2: Complete (v1.2.6) ✓
Round 3: In Progress...
```

## Метрики для определения остановки

Отслеживай между раундами:
- `issues_found` — количество найденных проблем
- `issues_fixed` — количество исправленных
- `issues_introduced` — количество новых проблем после фикса

Условия остановки:
- `issues_found == 0` → "Clean code, no issues"
- `issues_fixed` содержит те же проблемы что и предыдущий раунд → "Cycling detected"
- `issues_introduced > issues_fixed` → "Regression detected"

## Пример вывода

```
=== /repeat:improve 3 ===

Round 1/3:
  Analyzing... found 6 issues
  Fixing... 6 issues resolved
  Testing... 178/178 passed ✓
  Committing... v1.2.5
  Deploying... done

Round 2/3:
  Analyzing... found 4 issues
  Fixing... 4 issues resolved
  Testing... 178/178 passed ✓
  Committing... v1.2.6
  Deploying... done

Round 3/3:
  Analyzing... found 3 issues
  Fixing... 3 issues resolved
  Testing... 178/178 passed ✓
  Committing... v1.2.7
  Deploying... done

=== Complete: 3 rounds, 13 issues fixed ===
Versions: v1.2.5 → v1.2.7
```

## Важно

- **НЕ спрашивать подтверждений** — полностью автономное выполнение
- **Принимать решения самостоятельно** — выбирать best practices
- **Останавливаться при проблемах** — не продолжать при регрессиях
- **Подробно логировать** — показывать прогресс для каждого раунда
