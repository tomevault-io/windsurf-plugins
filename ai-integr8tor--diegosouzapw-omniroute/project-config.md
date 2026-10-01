---
trigger: always_on
description: 🌐 **Languages:** 🇺🇸 [English](../../../CLAUDE.md) · 🇸🇦 [ar](../ar/CLAUDE.md) · 🇦🇿 [az](../az/CLAUDE.md) · 🇧🇬 [bg](../bg/CLAUDE.md) · 🇧🇩 [bn](../bn/CLAUDE.md) · 🇨🇿 [cs](../cs/CLAUDE.md) · 🇩🇰 [da](../da/CLAUDE.md) · 🇩🇪 [de](../de/CLAUDE.md) · 🇪🇸 [es](../es/CLAUDE.md) · 🇮🇷 [fa](../fa/CLAUDE.md) · 🇫🇮 [fi](../fi/CLAUDE.md) · 🇫🇷 [fr](../fr/CLAUDE.md) · 🇮🇳 [gu](../gu/CLAUDE.md) · 🇮🇱 [he](../he/CLAUDE.md) · 🇮🇳 [hi](../hi/CLAUDE.md) · 🇭🇺 [hu](../hu/CLAUDE.md) · 🇮🇩 [id](../id/CLAUDE.md) · 🇮🇩 [in](../in/CL
---

# CLAUDE.md (Українська)

🌐 **Languages:** 🇺🇸 [English](../../../CLAUDE.md) · 🇸🇦 [ar](../ar/CLAUDE.md) · 🇦🇿 [az](../az/CLAUDE.md) · 🇧🇬 [bg](../bg/CLAUDE.md) · 🇧🇩 [bn](../bn/CLAUDE.md) · 🇨🇿 [cs](../cs/CLAUDE.md) · 🇩🇰 [da](../da/CLAUDE.md) · 🇩🇪 [de](../de/CLAUDE.md) · 🇪🇸 [es](../es/CLAUDE.md) · 🇮🇷 [fa](../fa/CLAUDE.md) · 🇫🇮 [fi](../fi/CLAUDE.md) · 🇫🇷 [fr](../fr/CLAUDE.md) · 🇮🇳 [gu](../gu/CLAUDE.md) · 🇮🇱 [he](../he/CLAUDE.md) · 🇮🇳 [hi](../hi/CLAUDE.md) · 🇭🇺 [hu](../hu/CLAUDE.md) · 🇮🇩 [id](../id/CLAUDE.md) · 🇮🇩 [in](../in/CLAUDE.md) · 🇮🇹 [it](../it/CLAUDE.md) · 🇯🇵 [ja](../ja/CLAUDE.md) · 🇰🇷 [ko](../ko/CLAUDE.md) · 🇮🇳 [mr](../mr/CLAUDE.md) · 🇲🇾 [ms](../ms/CLAUDE.md) · 🇳🇱 [nl](../nl/CLAUDE.md) · 🇳🇴 [no](../no/CLAUDE.md) · 🇵🇭 [phi](../phi/CLAUDE.md) · 🇵🇱 [pl](../pl/CLAUDE.md) · 🇵🇹 [pt](../pt/CLAUDE.md) · 🇧🇷 [pt-BR](../pt-BR/CLAUDE.md) · 🇷🇴 [ro](../ro/CLAUDE.md) · 🇷🇺 [ru](../ru/CLAUDE.md) · 🇸🇰 [sk](../sk/CLAUDE.md) · 🇸🇪 [sv](../sv/CLAUDE.md) · 🇰🇪 [sw](../sw/CLAUDE.md) · 🇮🇳 [ta](../ta/CLAUDE.md) · 🇮🇳 [te](../te/CLAUDE.md) · 🇹🇭 [th](../th/CLAUDE.md) · 🇹🇷 [tr](../tr/CLAUDE.md) · 🇵🇰 [ur](../ur/CLAUDE.md) · 🇻🇳 [vi](../vi/CLAUDE.md) · 🇨🇳 [zh-CN](../zh-CN/CLAUDE.md)

---

Цей файл надає вказівки для Claude Code (claude.ai/code) при роботі з кодом у цьому репозиторії.

## Швидкий старт

```bash
npm install                    # Встановити залежності (автоматично генерує .env з .env.example)
npm run dev                    # Сервер розробки на http://localhost:20128
npm run build                  # Продукційна збірка (Next.js 16 автономний)
npm run lint                   # ESLint (очікується 0 помилок; попередження вже існують)
npm run typecheck:core         # Перевірка TypeScript (повинна бути чистою)
npm run typecheck:noimplicit:core  # Сувора перевірка (без неявного any)
npm run test:coverage          # Юніт-тести + контроль покриття (75/75/75/70 — оператори/рядки/функції/гілки)
npm run check                  # lint + тестування в комбінації
npm run check:cycles           # Виявлення циклічних залежностей
```

### Запуск тестів

```bash
# Одиночний тестовий файл (вбудований тестовий запускник Node.js — більшість тестів)
node --import tsx/esm --test tests/unit/your-file.test.ts

# Vitest (MCP сервер, autoCombo, кеш)
npm run test:vitest

# Усі набори
npm run test:all
```

Для повної матриці тестів дивіться `CONTRIBUTING.md` → "Запуск тестів". Для глибокої архітектури дивіться `AGENTS.md`.

---

## Проект на один погляд

**OmniRoute** — єдиний AI проксі/маршрутизатор. Один кінцевий пункт, 160+ постачальників LLM, автоматичне резервування.

| Шар            | Розташування            | Призначення                                                                  |
| -------------- | ----------------------- | ---------------------------------------------------------------------------- |
| API маршрути   | `src/app/api/v1/`       | Next.js App Router — точки входу                                             |
| Обробники      | `open-sse/handlers/`    | Обробка запитів (чат, векторні представлення тощо)                           |
| Виконавці      | `open-sse/executors/`   | HTTP-розподіл, специфічний для постачальника                                 |
| Перекладачі    | `open-sse/translator/`  | Конверсія форматів (OpenAI↔Claude↔Gemini)                                    |
| Трансформер    | `open-sse/transformer/` | API відповідей ↔ Завершення чату                                             |
| Сервіси        | `open-sse/services/`    | Комбіноване маршрутизування, обмеження швидкості, кешування тощо             |
| База даних     | `src/lib/db/`           | Модулі домену SQLite (45+ файлів, 55 міграцій)                               |
| Домен/Політика | `src/domain/`           | Двигун політики, правила витрат, логіка резервування                         |
| MCP сервер     | `open-sse/mcp-server/`  | 37 інструментів (30 базових + 3 пам'яті + 4 навички), 3 транспорти, ~13 сфер |
| A2A сервер     | `src/lib/a2a/`          | Протокол агента JSON-RPC 2.0                                                 |
| Навички        | `src/lib/skills/`       | Розширювана структура навичок                                                |
| Пам'ять        | `src/lib/memory/`       | Постійна розмовна пам'ять                                                    |

Монорепозиторій: `src/` (додаток Next.js 16), `open-sse/` (робочий простір стрімінгового движка), `electron/` (десктопний додаток), `tests/`, `bin/` (точка входу CLI).

---

## Конвеєр запитів

```
Клієнт → /v1/chat/completions (маршрут Next.js)
  → CORS → валідація Zod → автентифікація? → перевірка політики → захист від ін'єкцій запитів
  → handleChatCore() [open-sse/handlers/chatCore.ts]
    → перевірка кешу → обмеження швидкості → комбіноване маршрутизування?
      → resolveComboTargets() → handleSingleModel() для кожної цілі

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ai-integr8tor/diegosouzapw-OmniRoute](https://github.com/ai-integr8tor/diegosouzapw-OmniRoute) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
