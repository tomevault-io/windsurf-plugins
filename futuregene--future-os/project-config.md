---
trigger: always_on
description: FutureOS desktop app: Tauri + React + TypeScript, frontend `src/`, Tauri backend `src-tauri/` (Rust), connects to the repo-root agent via gRPC. For overall monorepo architecture/build, see **repo-root `CLAUDE.md`**; this file covers `desktop/` only.
---

# Desktop Development Guide (`desktop/`)

FutureOS desktop app: Tauri + React + TypeScript, frontend `src/`, Tauri backend `src-tauri/` (Rust), connects to the repo-root agent via gRPC. For overall monorepo architecture/build, see **repo-root `CLAUDE.md`**; this file covers `desktop/` only.

## Document Map (read relevant sections on demand — don't pull whole files into context)

> Development docs live under `docs/internals/desktop/` (repo-root-relative paths below; formerly `desktop/DEV_MD/`).

| Document | Content | When to Read / Modify |
|---|---|---|
| `docs/internals/desktop/PRODUCT.md` (~35KB) | Product positioning, module boundaries, workspace object semantics, desktop experience | **Read** when changing product behavior / adding features / confirming domain semantics; **modify** only when product decisions change |
| `docs/internals/desktop/ER.md` (~42KB) | Data objects & relationships, table inventory, schema design decisions | **Read** when changing store / data flow; **modify** and keep in sync when schema changes |
| `docs/internals/desktop/COLOR.md` (~5KB) | Color semantic tokens + quick usage reference | **Read** when picking colors / changing styles; **modify** only when adding/changing tokens |
| `docs/internals/desktop/SANDBOX/COMMON.md` | Shared rules, tiers, approval UI/protocol, decisions and Codex references | **Read** for approval semantics; distinguish implemented behavior, accepted limitations and future plans |
| `docs/internals/desktop/SANDBOX/MACOS.md` / `LINUX.md` / `WINDOWS.md` | Platform implementation, differences, diagnostics, progress, validation procedures and evidence | **Read** the relevant platform; historical PASS is not validation of a new candidate; preserve the accepted Windows unelevated and Linux snapshot boundaries |
| `docs/internals/desktop/CONTEXT_COMPACTION.md` / `docs/internals/desktop/CONNECTION.md` | Compaction plans / remote product rationale, architecture, connection contract and implementation plan | **Read** for the corresponding feature; verify plan-vs-current against code |
| `docs/internals/desktop/embedded-terminal.md` | Embedded terminal: architecture, wire protocol, security model, lifecycle, platform status | **Read** before touching `src-tauri/src/terminal/` or `src/features/terminal/`; **modify** when the protocol or its boundaries change |

> `docs/internals/desktop/PRODUCT.md` / `docs/internals/desktop/ER.md` are large: use `Read` with `offset/limit` to read **specific sections** from the chapter index below — don't load the whole file.

### Chapter Quick Reference
- **PRODUCT.md**: §1 Positioning · §2 Module Boundaries · §3 Product Principles · §4 Work Objects (4.1 Workspace / 4.2 Chat / 4.3 Message / 4.4 Run / 4.5 Tool / 4.6 Approval / 4.7 Review / 4.8 Artifact / 4.9 Research / 4.10 Data / 4.11 Skill / 4.12 Attachment) · §5 Desktop Experience (5.1 Three-panel / 5.2 Left Nav / 5.3 Chat Area / 5.4 Right Context / 5.5 Colors / **5.6 Settings: Provider/Model/Login**) · §6 Agent Workflow · §9 Roadmap
- **ER.md**: §2 Relationship Overview · §3 Naming Conventions · §4 Objects (4.1 Workspace … 4.8 Approval Request / 4.9 Review Changeset / 4.10 Review File Change (incl. **Shadow Review extension**: `review_snapshots` table + changeset/file_change extension columns) … 4.20 Object Reference) · §5 V1 Table Inventory · §6 Key Design Decisions (**6.8 Shadow Repo "Previous Change Set"** / **6.9 Provider/Model/Login Config**)

> **Shadow Review** (run-level "previous change set"): product semantics in docs/internals/desktop/PRODUCT.md §4.7, data model in docs/internals/desktop/ER.md §4.10, design tradeoffs in docs/internals/desktop/ER.md §6.8. Read all three before modifying shadow repo / snapshot / changeset code (`src-tauri/src/shadow_review/`, `store/review_snapshots.rs`).

> **Provider / Model / FutureGene Login**: product behavior in docs/internals/desktop/PRODUCT.md §5.6, storage & login implementation in docs/internals/desktop/ER.md §6.9, custom-provider field validation in `agent_providers/validate.rs` (frontend mirror: `CustomProviderDialog.tsx` + settings.json strings `idPattern`/`idLength`/`baseUrlInvalid`). Read these before modifying `agent_providers/` (Providers view + custom-provider upsert/delete) / `future_platform.rs` (platform / model-API URL resolution, shared by login/skills/debug) / `auth_store.rs` / `future_login.rs` / `commands/login.rs`.

## Code Structure (`src/`)

- `components/layout/` — `AppShell` (layout orchestration) + `ContextPanel` + `ActivityRail` (left nav) + dialog shells (`AppShellDialogs` / `WorkspaceDialogs` / `LeftPanelTitlebarToggle`); `hooks/` contains AppShell domain hooks: `useThreadStore` / `useAgentConnection` / `useApprovals` / `useAppSettings` / `useModelSelection` / `useNewConversation` (new-conversation create flow: pending prompt + `startNewConversation`) / `useThreadDialogs` / `useUnreadThreads` / `useWorkspaceDialogs` / `useDropUpMenu`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [futuregene/future-os](https://github.com/futuregene/future-os) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
