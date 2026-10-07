---
trigger: always_on
description: This file is the repository's short map. The detailed source of truth lives in
---

# Prometeu — guide for agents

This file is the repository's short map. The detailed source of truth lives in
`ARCHITECTURE.md` and in `docs/`; do not copy whole documents here.

## Before changing code

1. Read `ARCHITECTURE.md` to locate the affected boundary.
2. Open only the `docs/` documents pointed to for that area.
3. Confirm the existing behavior in the tests and in the code.
4. Preserve local changes that do not belong to the task.

If code and documentation diverge, use the code's verifiable behavior to
diagnose the situation and update the documentation in the same change. Do not
turn a divergence into a silent architectural decision.

## Documentation map

- `README.md`: product, visible behavior and quick start.
- `CONTRIBUTING.md`: contribution workflow and links to implementation recipes.
- `ARCHITECTURE.md`: context, containers, responsibilities and core flows.
- `docs/architecture/`: details of the flows and dependency rules.
- `docs/contracts/`: formats that cross processes or layers.
- `docs/decisions/`: architectural decisions; proposals are not rules in force.
- `docs/quality/`: support matrices and verification strategy.
- `docs/operations/`: development, CI and release.

Start at `docs/README.md` for the complete index.

## Product invariants

- A session is the transcript; the agent's process may die and be resumed.
- Claude and Codex are external adaptations. Protocol differences must stay at
  the edge, without spreading vendor payloads through the presentation.
- The workspace stage is the person's decision; the tab's status is the agent's
  observed state. Do not mix the two.
- The shared session still runs only on the owner's Mac. The relay coordinates
  and persists ciphertext and metadata; content uses E2EE v4 with trust on first
  contact and automatic adoption of changed keys. The limits are in ADRs 0022
  and 0042.
- The relay's source of types and validation is `relay/src/protocol.ts`.
- Visible interface text goes through i18n. User data and agent output stay in
  their original language.
- UI controls reuse `src/ui.ts`, `src/menu.ts` and the shared tokens.
  See the [Design System](docs/architecture/design-system.md) before creating or
  changing controls.
  Find Desktop components, stories and real consumers in
  [`src/components/`](src/components/README.md) and its `catalog.json`.
  For isolated Desktop screens, follow the [composition recipe](docs/architecture/desktop-composition.md).

The complete dependency rules are in
`docs/architecture/dependency-rules.md`.

## Where each responsibility lives

- `src/agents.ts`: typed catalog, model/provider association and capabilities.
- `src/timeline.ts`: pure reducer from the conversation stream to screen items.
- `src/chat.ts`: presentation and interaction of the conversation.
- `src/desk.ts`: the desk, the home screen — one `ChatView` per running
  conversation.
- `src-tauri/crates/core/`: portable board models, conversation ordering,
  session, launch, account and tool rules behind injected ports (ADR 0085).
- `src-tauri/crates/process/`, `profiles/`, `tools/`, `files/`, `git/`,
  `protocols/` and `oauth/`: Tauri-free native adapters shared by the desktop
  and the WSL runtime.
- `src-tauri/src/main.rs`: desktop composition root; `chat.rs`, `chat/host.rs`
  and `session/launch.rs` compose conversations from core services.
- `src-tauri/src/claude.rs`, `antigravity.rs` and `crates/protocols/src/codex.rs`:
  provider adapters; `src-tauri/src/agent_launch.rs` registers native preparation.
- `src-tauri/src/session.rs`: desktop use cases and workspace/tab lifecycle;
  `state.rs` and `board_store.rs` adapt board events and file storage.
- `src-tauri/src/accounts.rs` and `account_login.rs`: desktop account facade and
  native authentication.
- `src-tauri/crates/runtime/`: Unix execution host used by Windows through WSL.
- `src-tauri/crates/bridge/`, `crates/wsl-desktop/` and `src/windows/`: native
  Windows shell that loads the existing interface over the WSL runtime (ADR 0084).
- `src/team.ts`: the collaboration desktop shell (team.json, organizations,
  Tauri ports, facade).
- `src/team-member.ts` and `src/team-*.ts`: the portable collaboration core and
  its features; they never import Tauri or IPC (ADR 0026).
- `src/mobile/`: the browser shell over the same core, vendored in the Cloud as
  a bundle (ADR 0028).
- `src-tauri/src/evaluation.rs` and `src-tauri/src/typesafe.rs`: optional
  context-evaluation port and its TypeSafe adapter (ADR 0058);
  `src/context-review.ts` holds the missing-context review rules.
- `relay/src/protocol.ts`: the network contract shared by the app and the
  Worker.
- `relay/src/logic.ts`: the relay's pure rules.

## Contracts and decisions

A change in a persisted format, IPC, the relay, the conversation protocol, a
trust boundary or a dependency between layers requires:

- updating the corresponding contract in `docs/contracts/`;
- adding or updating an ADR when there is a choice with trade-offs;
- including a compatibility test or explaining why it does not apply.

Keep ADRs aligned with the implemented decision. When replacing or removing a
decision, consolidate the guarantees still in force into its replacement and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [prometeucorp/prometeu](https://github.com/prometeucorp/prometeu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
