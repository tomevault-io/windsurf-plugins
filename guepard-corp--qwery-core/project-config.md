---
trigger: always_on
description: You are contributing to **qwery-agent**, an AI data analyst with a TUI.
---

# AGENTS.md — Strict rules for AI contributors

You are contributing to **qwery-agent**, an AI data analyst with a TUI.
This file is the **machine-oriented contract**. README.md is for humans.

If you don't read this end-to-end, you will violate an invariant.
Decisions live in `memory/ADR/decisions.md` and `memory/ADR/ui-decisions.md`.
Architecture diagrams live in `memory/diagrams/`.

---

## 1. Non-negotiable invariants

These hold **before** anything else. Breaking them blocks a PR.

1. **Privacy boundary (ADR #28).** The LLM **never** receives row-level data from any datasource.
   - `runQuery` accepts aggregate-only SQL, single-row scalar output, validated locally.
   - `schema` and `describeQuery` return only column metadata.
   - `present` renders results **locally** via Mustache + helpers — only `{ ok, rowCount }` goes upstream.
   - The privacy invariant test (`tooling/privacy-check.ts`) is enforced in pre-push and CI.
2. **Tool minimalism (ADR-locked).** Do not add a tool to the core agent unless it materially improves LLM reasoning. "Convenience" is not a justification.
3. **Hexagonal architecture (ADR #12).** `domain/` and `application/` never import from `adapters/` or `extensions/`. Enforced by `dependency-cruiser`.
4. **English only.** All code, comments, identifiers, commit messages and documentation are in English. Chat with the user can be in other languages; the project file content cannot.
5. **No env vars for LLM config.** Providers are configured via `/models` in the TUI, stored at `~/.qwery/config.json`. Never re-introduce env-var fallbacks for provider credentials.
6. **No code without unit tests.** Every new module / feature / non-trivial change ships with `bun:test` coverage **in the same turn**. A code change without tests is incomplete. Cover happy path + at least one edge case + at least one failure mode. For LLM-dependent code, inject a stream function or use Vercel AI SDK's `simulateReadableStream` / `simulateStreamingMiddleware` rather than calling real models.

---

## 2. Code quality rules

- **Latest dependencies.** When in doubt, use the latest stable. Pin only when an upstream breakage is known.
- **TypeScript strict.** No `any`, no `// @ts-ignore` without an immediately adjacent justification comment.
- **No dead code.** Remove unused imports, exports, functions immediately.
- **Comments are scarce.** Comment only the *why* (constraints, invariants, surprising decisions). Never comment what the code already says.
- **Errors are typed.** Throw `Error` subclasses or return `{ ok: false; error: ... }`. Never `throw 'string'`.
- **No emojis in code or commits** unless the user explicitly requests them. Emojis used in TUI rendering (`🔒`, `✓`) are intentional and documented.
- **Logger over `console`.** Use `src/lib/logger.ts` for diagnostics. `console.*` is reserved for the diagnostic `scripts/`.

---

## 3. Hexagonal layout (target after restructure)

```
qwery-agent/
├── apps/cli/                  # Ink TUI — primary adapter
├── packages/
│   ├── domain/                # entities, value objects, ports (interfaces)
│   ├── application/           # use cases orchestrating domain via ports
│   ├── adapters/
│   │   ├── llm-aisdk/         # Vercel AI SDK adapter
│   │   ├── compute-duckdb/    # DuckDB adapter
│   │   ├── branching-gfs/     # GFS adapter
│   │   └── ui-ink/            # Ink renderers (if extracted from apps/cli)
│   ├── extension-sdk/         # public contract for third-party extensions
│   └── extensions/            # first-party extensions (csv, json, sqlite, mysql, postgres, …)
├── tooling/                   # build / lint / dep-cruiser / privacy-check
└── memory/                    # ADRs + diagrams
```

Rules:
- A package may **only** import from its own layer and the layers it depends on. `domain` is at the bottom; `adapters/*` and `extensions/*` are at the top.
- Each adapter is one package. One package per concern. Independent versioning via Changesets.
- New artifact types (Report, Querybook, …) ship as **extensions**, not as core code (ADR #26).

### Use cases (ADR #33)

- **Location**: `packages/application/use-cases/<aggregate>/<operation>.ts`. **Never** in `packages/domain/` — domain stays free of orchestration and I/O.
- **One file, one unit**. A use case is one class OR one function — not both, not split across an interface file and an implementation file.
- **Shape — class form**:
  ```ts
  export class CreateConversation {
    constructor(private readonly deps: CreateConversationDeps) {}
    async execute(input: CreateConversationInput): Promise<CreateConversationOutput> { /* ... */ }
  }
  ```
- **Shape — functional form** (preferred when stateless):
  ```ts
  export interface CreateConversationDeps { conversationRepo: ConversationRepository }
  export async function createConversation(
    deps: CreateConversationDeps,
    input: CreateConversationInput,
  ): Promise<CreateConversationOutput> { /* ... */ }
  ```
- **Naming**: the verb-noun name is the symbol (`CreateConversation`, `createConversation`). No suffix `Service`, `Handler`, `Manager`, `UseCase`.
- **Dependencies**: use cases depend on **domain ports**. Never on concrete adapters. The composition root (`apps/cli/src/main.tsx`) wires real adapters into the deps.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Guepard-Corp/qwery-core](https://github.com/Guepard-Corp/qwery-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
