# TODOS

## Critical — нужно до первого запуска

### TODO-1: Таймаут Claude API + error UX
**What:** AbortController с 30-секундным таймаутом на anthropic.messages.stream(), catch-блок закрывает SSE и шлёт `event: error` клиенту, toast в chat.html.
**Why:** Без этого пользователь видит бесконечный спиннер при зависании Claude или 529 (overloaded). Failure полностью silent.
**Pros:** Без этого первый реальный пользователь не пройдёт онбординг при нагрузке.
**Cons:** Минимальные — 10-15 строк кода.
**Context:** Выявлено в /plan-eng-review 2026-04-13. Backend ещё не написан — добавить в server.js при реализации POST /api/chat.
**Where to start:** `server.js`, функция обработки POST /api/chat, блок stream.
**Depends on:** Backend (server.js) должен существовать.

---

### TODO-2: Валидация JSON от Claude на финальном шаге
**What:** Когда чат заканчивается, сервер просит Claude сформировать финальный JSON для potok.html. Нужно: regex-вырезка из markdown-блока (Claude часто оборачивает JSON в ```json```), ручная/zod валидация структуры, retry с уточняющим промптом если JSON невалидный.
**Why:** Без этого JSON.parse бросит exception, модель не сохранится в БД, пользователь видит пустую страницу вместо воронки. Именно здесь заканчивается весь флоу — самое критичное место.
**Pros:** Без retry продакшн не выживет — Claude не всегда даёт чистый JSON.
**Cons:** Retry добавляет задержку (ещё 5-10 сек) и один лишний API call при failure.
**Context:** Выявлено в /plan-eng-review 2026-04-13. JSON-контракт: `{company, channels[], funnel_stages[], insights[]}`.
**Where to start:** `server.js`, функция финализации модели (вызывается когда `session.complete === true`).
**Depends on:** TODO-1 (таймаут) должен быть готов первым.
