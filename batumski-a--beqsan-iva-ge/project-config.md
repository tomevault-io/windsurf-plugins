---
trigger: always_on
description: You are working on **BEQSAN**, an online platform for an aluminum & PVC door/window manufacturer based in Salibauri, Batumi.
---

# BEQSAN — Claude Code Operating Instructions

You are working on **BEQSAN**, an online platform for an aluminum & PVC door/window manufacturer based in Salibauri, Batumi.

- **Owner:** Roman Sharashidze (BEQSAN LTD)
- **Built by:** IVA (Lasha)
- **Hosting:** `beqsan.iva.ge` (public), `admin.beqsan.iva.ge` (admin), `api.beqsan.iva.ge` (API)
- **Source of truth for product vision:** [docs/kickoff.md](docs/kickoff.md)

## Workspace layout

This workspace uses a **flat split**, not a single monorepo root:

```
e:\BEQSAN.IVA.GE\
├── BACK/        # .NET 8 solution (BEQSAN.sln, src/, tests/, docker-compose.yml)
├── FRONT/       # Vite + React 18 + TypeScript SPA (public site, eventually also admin app)
├── docs/        # kickoff, ADRs, schema, API docs, questions
├── .claude/     # skill library + slash commands
└── CLAUDE.md    # this file
```

All .NET paths live under `BACK/`. All frontend paths live under `FRONT/`. Shared docs and Claude config live at the workspace root.

## How to use this codebase

Before any non-trivial task, read the relevant skill file in `.claude/skills/`. If unsure which skill applies, read [.claude/skills/INDEX.md](.claude/skills/INDEX.md).

- For **any UI work** → read `design-system/SKILL.md` **first**. No exceptions.
- For **any backend work** → read `dotnet-clean-arch/SKILL.md` **first**.
- For **configurator work** → read both `configurator-architecture/` and `3d-scene-design/`.
- For **AI features** (photo measurement, room render) → read `ai-integration/SKILL.md`.
- For **any user-facing copy** → read `georgian-ux/SKILL.md` and `content-voice/SKILL.md`.

When a skill applies, follow it. When two skills conflict, follow the more specific one and note the conflict in `docs/questions.md`.

## Architecture invariants (NEVER violate)

- **Backend:** Clean Architecture, .NET 8, MediatR + `Result<T>` pattern. Domain depends on nothing. Application depends on Domain only. Infrastructure can reference Domain + Application. Api sits on top.
- **Frontend:** React 18 + TypeScript **strict** mode. **NO** `any`. **NO** `@ts-ignore` (use `@ts-expect-error` with a reason if you must, and only as a last resort).
- All **public catalog reads** go via **Dapper**. All **admin writes** go via **EF Core**. Never mix in the same handler.
- Every endpoint returns `Result<T>` mapped to HTTP via a shared `ToActionResult()` extension. **No raw exceptions cross the API boundary** — they become `Result.Failure` and get mapped centrally.
- **Money** is `decimal(18,2)` via a `Money` value object. Never `double`/`float`.
- **Dimensions** in centimeters, stored as `int`. Never store as decimal/double.
- **Phone numbers** in E.164 format (`+995595XXXXXX`). Normalize at the edge (input handlers, SMS receivers).
- **Times** in UTC at rest; converted to `Asia/Tbilisi` only at the presentation edge.
- **Currency** is GEL (`₾`), formatted with Georgian space thousand separator + comma decimal: `1 234,56 ₾`.

## Code style

- **Backend:** nullable enabled, file-scoped namespaces, primary constructors where natural, `sealed` records for DTOs/Commands/Queries, `internal sealed` for handlers, public API surface only when needed.
- **Frontend:** function components only, named exports preferred over default, `const Component = () => ...` over `function Component()` for consistency with Tailwind/typed-children helpers, props typed inline if local — `type Props = { ... }` colocated.
- **Tailwind:** no inline `style` attributes, always use the `cn()` utility for conditionals, design tokens come from `tailwind.config.ts`. No magic Tailwind values — extend the theme instead.
- **Commits:** Conventional Commits (`feat:`, `fix:`, `chore:`, `refactor:`, `docs:`, `perf:`, `test:`, `build:`, `ci:`).
- **Branches:** `type/scope-short-description` (e.g. `feat/configurator-step-3`, `fix/order-tracking-redirect`).

## What to NEVER do

- Never expose admin endpoints without `[Authorize]` and explicit permission check.
- Never log PII (phone, email, full name) at Information level. Use Warning+ only, with masked values: `Log.ForContext("PhoneHash", Hash(phone))`.
- Never reproduce supplier names or supplier-cost pricing in **public** copy or **public** API responses. Internal admin views only.
- Never use **lorem ipsum** — write real Georgian copy. Empty-state copy has personality (see `content-voice/SKILL.md`).
- Never default to `Inter`, `Roboto`, `Open Sans`, or any other generic web font. Use the project font stack: `BPG Glaho Sans` / `BPG Mrgvlovani Caps` / `FiraGO` / `JetBrains Mono`.
- Never write English UI copy as a placeholder. Georgian is primary; en/ru come via i18next once the Georgian copy is approved.
- Never introduce a third-party dependency without a one-line justification in the PR description.
- Never use `@ts-ignore`, `as any`, `// eslint-disable` without a specific rule name + comment explaining why.
- Never commit secrets. Use `.env`, `dotnet user-secrets` (dev), Azure Key Vault or equivalent (prod).

## Languages

- **All user-facing copy:** Georgian (primary). Translations to `en` and `ru` come via i18next once the Georgian source is finalized.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Batumski-A/BEQSAN.IVA.GE](https://github.com/Batumski-A/BEQSAN.IVA.GE) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
