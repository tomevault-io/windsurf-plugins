---
trigger: always_on
description: These instructions apply to humans and coding agents.
---

# pi-forge repository guidance

These instructions apply to humans and coding agents.

## Current mode

pi-forge is continuing the lean 0.5 line through the accepted amendments in [docs/design/architecture-0.5.md](docs/design/architecture-0.5.md), including the authorized [capability lanes and accepted 9/21 amendment](docs/design/pi-forge-system-update-design-notes.md) and the Pi 0.87 migration. Functional source is delivered across all planned lanes: the codec/event foundation, human CLI core (`/capability`), plain metadata anchor projection with ordinal materialization, non-modal Current session controls/projection workspace, live Preset bindings (`capabilities`) in a dedicated peer Capability bindings tab with finite overrides and opt-in `modelCallable: true`, restricted Agent control (`forge_capability` with list/status/enable/disable, ID ≤128 chars), Web Capabilities CRUD with SDK-grouped tool picker and `sourceRevision` stale-save protection, custom default tools policy (`tools.initial?: string[]`), and guarded human Web activation (`GET /api/capability-state/available`, `POST /api/capability-state/enable` with guard and fingerprint checks).

Accepted 9/21 amendment boundaries:
- Authorized 9/26 narrow creation follow-up: New Preset offers Default Pi prompt (existing default), Empty (`items: []`, no tool/skill policy; existing compiler fallbacks remain), and Minimal Worker (matches `examples/minimal-prompt-stack.json`). Only New has this choice; Import/Fork use their existing source. Suggested names follow template changes only before user input. No persisted template field, runtime/compiler change, broad layout redesign or release authorization.
- Breaking pre-release Capabilities rename: Instruction modes are now Capabilities. Use `/capability` with `add`, `list`, `bindings`, `enable`, `enable-bound`, `disable`, `status`, `reset`, and `help`; the Agent tool is `forge_capability` with `list`, `status`, `enable`, and `disable`. Existing development configs must convert to `capabilities` resources and new sessions; old directories, schemas, aliases, and continuing capability state are not supported. This is not a published compatibility claim.
- `tools.initial?: string[]` requires concrete valid tool names without wildcards. Omission preserves exact legacy behavior (selective allow selects registered matches; unrestricted/deny retains or filters the session baseline); `[]` configures zero default tools. The allow/deny ceiling remains unchanged and exclusive; mod/extension tools are added as allowed registered inactive. Configured defaults serve as the base while the preset is active (not a one-time reset per turn); disable/reset returns to defaults plus remaining capabilities; preset disable restores the reconciled session baseline.
- Capability grouped tool picker uses SDK `sourceInfo` / loaded packages or entry points and saves exact concrete tool names (no persisted package references, no auto-installed packages, and newly introduced tools are not auto-added). Inactive registered tools are visible; unloaded tools are unavailable in the session, but manual saved references are preserved.
- Preset bindings are located in a dedicated peer Capability bindings tab; Preset metadata no longer contains bindings; overrides and source-effective preview are collapsed under an advanced toggle.
- Default tools editor is opt-in, collapsible, and searchable, with advanced literal/wildcard allow/deny policy retained.
- Saving a capability updates library definitions and does not activate it. Saving an inactive Preset does not select or activate it; saving the active Preset immediately reloads and synchronizes its live tool and capability authorization policy without replacing frozen active capability snapshots (UI docs state actual behavior; do not claim all saves never change execution).
- New configs require Forge 0.5.5; older Forge ignores `tools.initial` (not downgrade-compatible). The main-package 0.5.5 baseline and 0.5.6 regression patch are published; 0.5.7 follows its own release gates. Minimum SDK 0.87 remains unchanged (pinned to `0.87.0`, peer range `>=0.87.0 <0.88.0`).
- Prompt caching: compatible Codex transports can retain request prefixes for first-time tool additions when retained history has no removals/redeclarations. Removal/re-addition uses full-current-tool serialization; no cache-hit or permission-bypass guarantee.
- WebUI first-pass refinement (9/23): compact resource identity/actions, content-first item editing, permission/default cards, progressive Regex editors, and sidebar/wide/focused inspection. Dirty Presets must be explicitly saved before Activate; no silent combined save-and-activate. Global Session controls retain their refresh/guard behavior. View-only expansion never dirties a resource. Settings retain queued autosave; the new indicators do not create another persistence owner. This is an implemented iteration for user testing, not a frozen release or new backend architecture.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MacroSony/pi-forge](https://github.com/MacroSony/pi-forge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
