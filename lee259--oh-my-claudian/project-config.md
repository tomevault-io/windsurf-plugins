---
trigger: always_on
description: Claudian is an Obsidian plugin that embeds provider-backed coding agents in a sidebar and inline-edit flow. Claude is the default provider. The built-in selectable providers are Claude, Codex, Cursor, Grok, OMP (Oh My Pi), OpenCode, and Pi; ACP is shared transport infrastructure, not a selectable provider. All providers plug into the same conversation model through `Conversation.providerId` and opaque provider-owned `providerState`.
---

# AGENTS.md

## Project

Claudian is an Obsidian plugin that embeds provider-backed coding agents in a sidebar and inline-edit flow. Claude is the default provider. The built-in selectable providers are Claude, Codex, Cursor, Grok, OMP (Oh My Pi), OpenCode, and Pi; ACP is shared transport infrastructure, not a selectable provider. All providers plug into the same conversation model through `Conversation.providerId` and opaque provider-owned `providerState`.

Do not assume provider parity. Check each provider's `capabilities.ts`, `registration.ts`, and UI config before wiring shared behavior.

## Scope Guides

- Before editing a scoped area, read its nearest scoped guide:
  - `src/app/AGENTS.md`
  - `src/core/AGENTS.md`
  - `src/features/AGENTS.md`
  - `src/features/chat/AGENTS.md`
  - `src/features/chat/execution/AGENTS.md`
  - `src/features/chat/tabs/AGENTS.md`
  - `src/providers/AGENTS.md`
  - `src/providers/acp/AGENTS.md`
  - `src/providers/claude/AGENTS.md`
  - `src/providers/codex/AGENTS.md`
  - `src/providers/cursor/AGENTS.md`
  - `src/providers/grok/AGENTS.md`
  - `src/providers/omp/AGENTS.md`
  - `src/providers/opencode/AGENTS.md`
  - `src/providers/pi/AGENTS.md`
  - `src/style/AGENTS.md`

## AGENTS.md Maintenance

- AGENTS.md is execution context for agents, not general documentation. Keep only repository- or scope-specific information that a capable agent would not reliably know; every statement must change implementation, review, or verification behavior.
- Keep repository-wide rules here; put local ownership, dependencies, invariants, failure modes, verification, and active decisions in the narrowest scoped guide that governs them.
- Do not duplicate inherited guidance or silently contradict it. State a necessary local exception and its rationale explicitly.
- Omit tours, ordinary implementation details, temporary status, and general engineering advice.
- Record a decision only when it is active, surprising from the code, expensive to reverse, and reflects a real tradeoff. State the decision, rationale, and any concrete reconsideration condition; use Git history as the archive.
- `CLAUDE.md` files should import the nearest `AGENTS.md`; do not duplicate shared guidance there.

## Commands

```bash
pnpm run dev
pnpm run build
pnpm run typecheck
pnpm run lint
pnpm run lint:fix
pnpm run test
pnpm run test:watch
pnpm run test:coverage
```

The default full check is:

```bash
pnpm run typecheck && pnpm run lint && pnpm run test && pnpm run build
```

Tests mirror `src/` under `tests/unit/` and `tests/integration/`.

## Architecture

Scoped guides define the source of truth and allowed mutators for state in their area.

| Area | Responsibility |
| --- | --- |
| `src/main.ts` | Plugin lifecycle and concrete application composition |
| `src/app/` | Application conversation, settings, provider-host, and storage services |
| `src/core/` | Provider-neutral runtime, registry, storage, tool, and type contracts |
| `src/providers/acp/` | Shared ACP transport, interaction, and session primitives without provider policy |
| `src/providers/*/` | Provider adaptors, provider-owned runtime protocol, history, storage, settings, and UI |
| `src/features/chat/` | Sidebar chat orchestration against provider-neutral contracts |
| `src/features/inline-edit/` | Inline edit modal and provider-backed edit services |
| `src/features/settings/` | Shared settings shell and provider tab assembly |
| `src/shared/` | Reusable UI components |
| `src/style/` | Modular CSS built into `styles.css` |

### Dependency Direction

In the rules below, `A -> B` means `A` may import or call `B`:

```text
composition root (`src/main.ts`) -> app services + features + provider registrations + core
app services -> core contracts
features -> FeatureHost + core contracts + shared UI
providers -> ProviderHost + core contracts + shared provider and UI primitives
```

- `core/` must not import feature code, app composition, or provider implementations.
- Feature code must not import provider implementations. Resolve provider behavior through core registries and contracts.
- Provider runtime and protocol code must not import chat views, feature controllers, or other feature orchestration.
- Existing Claude compatibility re-exports that point into `src/app/` are migration seams, not an allowed general dependency direction. Do not add new provider-to-app imports; move shared contracts into `core/` when touching those seams materially.
- `src/providers/acp/` may contain protocol primitives shared by ACP providers. Provider-specific launch policy, extensions, normalization, history, and state remain in the owning provider.
- If a dependency does not fit these directions, introduce or extend an explicit contract at the owning boundary instead of reaching across layers.

### Cross-Layer Ownership


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lee259/oh-my-claudian](https://github.com/lee259/oh-my-claudian) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
