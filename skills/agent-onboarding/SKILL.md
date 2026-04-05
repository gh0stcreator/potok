---
name: agent-onboarding
description: >
  Генерирует полный набор инструкций для нового агента The Pattern Industries:
  AGENTS.md, SOUL.md, HEARTBEAT.md. Используй при найме нового агента совместно
  с paperclip-create-agent.
---

# Скилл: agent-onboarding

Используй этот скилл после создания нового агента через `paperclip-create-agent`.
Он генерирует персонализированные инструкции на основе роли агента.

## Входные данные

Перед запуском собери:

- **Роль агента** — название (например: Copywriter, Researcher, Designer)
- **Зона ответственности** — 3–5 пунктов, что делает
- **Чего не делает** — граница с другими ролями
- **Принципы работы** — 3–5 специфичных для роли правил
- **Кому подчиняется** — кто менеджер
- **Голос** — тон и стиль (например: прямой, аналитический, creativeый)
- **agent_id** — UUID нового агента
- **instructions_path** — путь к директории инструкций агента

## Процедура

### Шаг 1. Прочитай общие документы

```bash
cat $PROJECT_ROOT/COMPANY.md
cat $PROJECT_ROOT/PRODUCT.md
cat $PROJECT_ROOT/TEAM.md
cat $PROJECT_ROOT/templates/AGENT-INSTRUCTIONS-TEMPLATE.md
```

### Шаг 2. Сгенерируй AGENTS.md

На основе шаблона (`templates/AGENT-INSTRUCTIONS-TEMPLATE.md`) и входных данных:

1. Замени все `[PLACEHOLDER]` конкретными значениями.
2. Секция "Контекст компании" — **без изменений**, всегда одинакова.
3. Принципы — только role-specific, не дублировать COMPANY.md.
4. Не включай продуктовый контекст inline — ссылайся на `PRODUCT.md`.

Сохрани в `{instructions_path}/AGENTS.md`.

### Шаг 3. Сгенерируй SOUL.md

Файл определяет голос и стиль агента. Структура:

```markdown
# SOUL.md — [Роль]

## Кто ты

[2–3 предложения об идентичности агента — не должности, а способе мышления]

## Голос и тон

- [Характеристика 1]
- [Характеристика 2]
- [Характеристика 3]

## Как ты общаешься

- В комментариях: [стиль]
- В задачах: [стиль]
- При эскалации: [стиль]

## Чего ты никогда не делаешь

- [Анти-паттерн 1 для этой роли]
- [Анти-паттерн 2]
```

Сохрани в `{instructions_path}/SOUL.md`.

### Шаг 4. Сгенерируй HEARTBEAT.md

Файл описывает рабочий процесс агента. Структура:

```markdown
# HEARTBEAT.md — [Роль]

Выполняй этот чек-лист в каждый heartbeat.

## 1. Контекст

- Прочитай AGENTS.md если первый запуск
- Проверь PAPERCLIP_TASK_ID, PAPERCLIP_WAKE_REASON

## 2. Инбокс

- GET /api/agents/me/inbox-lite
- Приоритет: in_progress → todo. Blocked — только если есть новый контекст.

## 3. Checkout и работа

- POST /api/issues/{id}/checkout перед любой работой
- 409 = задача занята, перейди к следующей

## 4. [Специфика роли]

[Добавь 3–5 шагов, специфичных для роли]

## 5. Отчёт

- PATCH status с комментарием: что сделал, почему, что дальше
- Если заблокирован: PATCH status=blocked с объяснением

## Правила

- Всегда X-Paperclip-Run-Id на мутирующих запросах
- Язык: русский
- Не дублируй — ссылайся на общие документы
```

Сохрани в `{instructions_path}/HEARTBEAT.md`.

### Шаг 5. Создай пустой TOOLS.md

```markdown
# TOOLS.md — [Роль]

(Добавляй заметки об инструментах по мере их освоения.)
```

Сохрани в `{instructions_path}/TOOLS.md`.

### Шаг 6. Зарегистрируй путь инструкций в Paperclip

```bash
curl -X PATCH "$PAPERCLIP_API_URL/api/agents/{agent_id}/instructions-path" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"path": "{instructions_path}/AGENTS.md"}'
```

### Шаг 7. Отчитайся

Прокомментируй задачу найма: что создано, пути к файлам, что агент должен прочитать в первый heartbeat.

## Интеграция с paperclip-create-agent

Запускай `agent-onboarding` **после** того как `paperclip-create-agent` вернул `agent_id`.
Передай `agent_id` и `instructions_path` на Шаг 6.

## Чек-лист результата

- [ ] `AGENTS.md` создан и содержит ссылки на COMPANY.md, PRODUCT.md, TEAM.md, ONBOARDING.md
- [ ] `SOUL.md` создан
- [ ] `HEARTBEAT.md` создан
- [ ] `TOOLS.md` создан
- [ ] Путь инструкций зарегистрирован через API
- [ ] Комментарий оставлен в задаче найма
