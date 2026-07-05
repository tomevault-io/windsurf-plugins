---
trigger: always_on
description: This repository is `GoldFXGraph`.
---

# AGENTS.md

## 1. Project Identity

This repository is `GoldFXGraph`.

GoldFXGraph is a full-stack gold research project for XAUUSD daily and near-real-time analysis.

The system should:

- read real XAUUSD market data
- fetch current/latest gold data when required
- compute deterministic technical indicators
- run explicit LangGraph multi-agent analysis
- produce structured forecast results
- persist research runs and forecasts
- display the forecast result in a Vue 3 + Tailwind frontend

This project is for research and decision support only.

Do not implement automatic trading, broker integration, real order execution, MCP, n8n, multi-model routing, full evaluation jobs, scorecards, or complex observability unless explicitly requested.

---

## 2. Repository Layout

Use a professional full-stack layout.

Expected structure:

```text
goldfxgraph/
├── AGENTS.md
├── README.md
├── pyproject.toml
├── docker-compose.yml
├── .env.example
├── openspec/
├── data/
│   └── raw/
├── src/
│   └── goldfxgraph/
├── tests/
└── apps/
    └── web/
        ├── package.json
        ├── index.html
        ├── vite.config.ts
        ├── tailwind.config.ts
        ├── postcss.config.js
        └── src/
            ├── main.ts
            ├── App.vue
            ├── router/
            │   └── index.ts
            ├── styles/
            │   └── main.css
            ├── pages/
            │   └── GoldForecastDashboard.vue
            ├── components/
            ├── services/
            ├── types/
            └── constants/
```

Backend application code must live under:

```text
src/goldfxgraph/
```

Backend tests must live under:

```text
tests/
```

Frontend application code must live under:

```text
apps/web/src/
```

Frontend global CSS must live under:

```text
apps/web/src/styles/main.css
```

OpenSpec files must live under:

```text
openspec/
```

Do not invent a different package structure unless the user explicitly approves it.

---

## 3. OpenSpec Rules

This project uses OpenSpec for non-trivial changes.

OpenSpec paths:

```text
openspec/specs/
openspec/changes/
openspec/changes/archive/
```

For any feature, architecture change, workflow change, database change, API change, or frontend module:

1. Read `AGENTS.md`.
2. Inspect existing code and OpenSpec files.
3. Create or update an OpenSpec change first.
4. Generate or update:
   - `proposal.md`
   - `design.md`
   - `tasks.md`
   - spec delta files
5. Stop for human review before implementation.
6. Apply the change only after approval.
7. Validate and archive after implementation.

Preferred OpenSpec commands:

```text
/spec propose <change-id>
/spec apply <change-id>
/spec archive <change-id>
```

If this project uses `/opsx:*` instead, use the equivalent OpenSpec commands.

Recommended first change:

```text
bootstrap-fullstack-xauusd-forecast-dashboard
```

This change should cover the initial backend workflow and the first Vue 3 forecast dashboard.

---

## 4. Superpowers Rules

This project uses Superpowers as the execution discipline.

OpenSpec defines what to build.  
Superpowers defines how to execute safely.

Use Superpowers skills explicitly when useful:

```text
brainstorming
writing-plans
test-driven-development
executing-plans
requesting-code-review
finishing-a-development-branch
```

During implementation:

1. Create an implementation plan from the OpenSpec change.
2. Implement one small task at a time.
3. Write or update tests where practical.
4. Run tests directly.
5. Fix failures directly.
6. Update `tasks.md`.
7. Review the diff before completion.

Do not ask the user to run tests manually unless credentials, permissions, or local services prevent the agent from running them.

---

## 5. Frontend Rules

The frontend uses:

```text
Vue 3
TypeScript
Vite
Tailwind CSS
```

Frontend root:

```text
apps/web/
```

Frontend global stylesheet:

```text
apps/web/src/styles/main.css
```

Use `ui-ux-pro-max` for frontend UI/UX design guidance or review before implementing dashboards, reports, layout, interaction, or visual design.

Frontend tasks should:

1. Clarify user flow and page purpose.
2. Produce a clean UI/UX plan before coding.
3. Keep visual design consistent.
4. Avoid generic placeholder UI.
5. Review final UI against the intended user experience.

Do not change frontend architecture or design system without approval.

---

## 6. Gold Forecast Dashboard Requirements

The main frontend page should show the gold forecast result.

Recommended page:

```text
apps/web/src/pages/GoldForecastDashboard.vue
```

The dashboard should display at least:

- current/latest XAUUSD price
- data timestamp
- data source
- daily open, high, low, close when available
- multi-agent analysis summary
- technical analysis summary
- macro or news analysis summary when available
- final forecast direction
- entry price
- take-profit price
- stop-loss price
- risk/reward ratio when available
- suggested holding period
- intraday action suggestion
- longer-term holding suggestion
- confidence score
- key risks
- disclaimer that this is research, not financial advice

Forecast direction should use structured values such as:

```text
bullish
bearish
neutral
```

Chinese UI labels can use:

```text
看多
看空
震荡/中性
```

Trading decision fields should be clearly separated from research explanation:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [michael-chow-arch/goldfxgraph](https://github.com/michael-chow-arch/goldfxgraph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
