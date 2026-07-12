---
trigger: always_on
description: **What this is:** a graph-based, durable, inspectable long-term project memory for AI coding assistants (Claude Code first). Local-first, user-owned, graph-first UI. See **`PLAN.md`** — it is the source of truth; read it before doing anything.
---

# Engram — context for Claude Code

**What this is:** a graph-based, durable, inspectable long-term project memory for AI coding assistants (Claude Code first). Local-first, user-owned, graph-first UI. See **`PLAN.md`** — it is the source of truth; read it before doing anything.

**Positioning (PLAN.md §1A):** Engram is the **reasoning/decision memory** layer (why/decided/what-bit-us) — *not* a code-structure graph (Graphify/CodeGraph). Those are complementary and can coexist. The validated wedge: an **editable, IDE-embedded graph pane** for reasoning memory + conflict surfacing + local — the closest concept (Cairn) has no UI/traction.

## Status (2026-07-11)
- **Shipped 2026-07-12 (v0.4.1):** pane fixes (settings menu scrolls with a viewport cap + contained overscroll; topbar z-index 10→12 so its dropdowns — settings/filter/search/burger — escape their stacking context and layer above open drawers); **write-verdict protocol** (write responses now also carry `suspects` — the look-alike pairs the write just queued, with an inline `action_required` instruction; `update_node_checked` returns them too; skill overhaul across all variants + agents blocks: decisions are captured unprompted since feature requests usually hide one, every write response is a verdict — matched → merge, warnings → check canon, suspects → judge NOW via resolve_suspect — and a genuine contradiction is the one audible exception to silent capture); **Subagents section** in the skills (subagents share the MCP connection and can search/brief but start cold and write under the parent session id — recall flows down via prompts, capture flows up via findings); **Windows release zip dropped** (release.yml ships only the raw .exe — install.ps1, self-update, and manual downloads all use it directly; nothing consumed the zip).
- **Phase 0 AND Phase 1 complete & verified** (121 tests: core 74, http 19, mcp 17, cli 11). `engram-alpha serve` serves the HTTP API + SSE + the embedded Vue pane at `http://127.0.0.1:8787`; release binary at `~/.cargo/bin/engram-alpha`. Latest published release: **v0.4.1** (2026-07-12). Release flow: push → `gh workflow run draft-release.yml -f version=X.Y.Z` → publish the draft (gh release edit --draft=false or the GitHub UI), which tags and builds artifacts. All crates share `[workspace.package] version` (0.4.1) which release.yml stamps from the tag — never version a crate individually; `claude-plugin/.claude-plugin/plugin.json` must carry the same version (a cli test enforces it — the plugin installs from the repo, not from release artifacts).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [techtheist/engram](https://github.com/techtheist/engram) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-12 -->
