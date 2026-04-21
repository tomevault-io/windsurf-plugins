---
trigger: always_on
description: **Context sources (in order of priority):**
---

# Copilot Workflow Instructions

## Core Principles

**Context sources (in order of priority):**
1. `specs/` — single source of truth (architecture, requirements, changelogs)
2. `./sample-project/` — read-only reference, never modify or run
3. **For complex tasks**: read 3 latest `specs/changelog/*.md` files

**Group by feature, not type.** Each feature = own subtree + own git repo.
Naming: `sharetribe-horizon-extensions-<name-separated-by-dash>`

---

## Folder Structure

### Client code

Group by feature under `src/extensions/<FeatureName>/`.

Example — Voucher / Discount feature:

```
.
├── src/
│   └── extensions/
│       └── Voucher/
│           ├── config/
│           │   ├── configA.js
│           │   └── configB.js
│           ├── components/
│           │   └── VoucherInputField.js
│           ├── containers/
│           │   └── EditVoucherPage/
│           │       ├── EditVoucherPage.js
│           │       └── EditVoucherPage.duck.js
│           ├── translations/
│           │   └── en.json
│           ├── validators.js
│           ├── routes.js
│           ├── reducers.js
│           ├── types.js
│           ├── api.js
│           └── README.md
```

### Server code

Group by feature under `server/extensions/<featureName>/`.

Example — Voucher / Discount feature (Voucherify integration):

```
.
└── server/
    └── extensions/
        ├── common/
        └── discount/
            ├── api/
            │   ├── index.js
            │   ├── get.js
            │   └── create/
            │       └── index.js
            ├── common/
            │   └── config/
            │       ├── stripe.js
            │       └── voucherify.js
            ├── middlewares/
            │   └── authentication.js
            └── mods/
                └── voucherify/
                    ├── customer/
                    │   ├── create.js
                    │   └── get.js
                    ├── voucher/
                    │   ├── get.js
                    │   └── create/
                    │       └── index.js
                    └── instance.js
```

### Common code

**Common:** `src/common/` (client) or `server/common/` (server) — shared between 2+ features. Start in feature folder, migrate to common when needed.

---

## Handling Complex Tasks

**For large, multi-feature tasks or architectural changes:**

Instead of trying to resolve immediately, trigger the agent-loop to handle systematically.

**Steps:**
1. **Assess complexity**: If task involves 3+ features, major refactoring, or spans multiple systems → use agent-loop
2. **Create iteration plan**: Write to `.task` file with:
   - Summary of overall goal
   - Breakdown by iteration (max 1 feature per iteration, 2 if tightly coupled for testing)
   - Each iteration: clear objective, files affected, acceptance criteria
3. **Trigger agent-loop**: Inform user to run `bash start-agent-loop.sh "$(cat .task)"`

**Iteration planning format (write to .task):**
```
Overall Goal: [Brief description]

Iteration 1: [Feature name]
- Objective: [What to build]
- Files: [List expected files]
- Tests: [What to verify]

Iteration 2: [Next feature]
- Objective: [What to build]
- Files: [List expected files]
- Tests: [What to verify]

[Continue for remaining features...]
```

**When to use agent-loop vs direct implementation:**
- **Direct**: Single feature, clear scope, < 5 files
- **Agent-loop**: Multiple features, architectural changes, cross-system integration, > 5 files

---

## Working Guidelines

**Before coding:**
1. **For complex tasks**: read 3 latest `specs/changelog/*.md` + `specs/architecture/*.md`
2. **If uncertain**: consult `specs/` (single source of truth)
3. Check `./sample-project/` for reference patterns (read-only, never modify)

**When implementing:**
- Design before coding (outline steps first)
- Keep scope small: 1-2 features max per iteration
- Add `// [context]` comments explaining *why*
- New user flows → create test in `specs/tests/`
- Commits: `feat|fix|chore(<scope>): <summary>`

**After changes:**
- Write changelog to `specs/changelog/YYYYMMDD_HHMMSS.md` with 4 sections (Update, New, Technical Debt, What Should Be Done Next) — skip for trivial edits
- Test if UI/API/auth changed or 3+ files touched
- State explicitly if testing/changelog skipped and why

---

## General Rules

- `./sample-project/` is read-only — never modify or run
- Write changelog for meaningful changes (skip trivial edits)
- Test scenarios mandatory for new flows
- Keep iterations small
- Comments explain *why*, not *what*

---

## Coding Conventions

### API Endpoints

#### Method usage

Use the correct HTTP method for each action — do not label everything `POST`.

| Method | Purpose |
|--------|---------|
| `GET` | Retrieve resources. Extra config via query params. |
| `POST` | Create resources only. |
| `PUT` | Replace an entire existing resource. |
| `PATCH` | Partially update an existing resource. |
| `DELETE` | Delete resources. Must be accompanied by authentication. |

#### Naming

Split endpoints into resource chunks. Use the HTTP method to express the action — the trailing segment should be one word. More than two words is a signal to split further.

✅
```
POST   /api/user       → create user
PUT    /api/user       → replace user
PATCH  /api/user       → partial update user

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Chimedeck/chimedeck](https://github.com/Chimedeck/chimedeck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
