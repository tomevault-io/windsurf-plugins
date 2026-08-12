---
trigger: always_on
description: This file captures only what cannot be inferred from the codebase itself.
---

# CLAUDE.md

This file captures only what cannot be inferred from the codebase itself.
## Rules for editing this file


Both developers and AI agents are expected to add entries as they encounter surprises.

- **Add an entry** when you encounter something unexpected: a build quirk, a non-obvious constraint, a dependency gotcha, or any behavior that would surprise the next agent or developer.
- **Add an entry** when a developer flags an anti-pattern produced by AI — describe the anti-pattern and the preferred alternative.
- **Do not** add codebase overviews, directory listings, or anything discoverable by reading the source.
- Keep entries concise: one line per lesson, grouped under a heading if a theme emerges.
- **Durable-invariant test** before adding a gotcha: "Would a future agent rediscover this in under 60 seconds via Grep + reading source?" If yes, it belongs in the git commit message, not here. Every entry costs tokens on every session — the bar for inclusion is **future agents will break things if they don't know this**. Per-fix narratives (test name, specific code path, fix story) go in PLAN-PHASE-5.md session notes instead.
- **Format rule (2026-06-10 trim 284→170 KB; 2026-07-06 trim 594→~280 KB after it silently regrew — do not regrow it)**: a new entry is 1–3 lines stating the trap/invariant and where to look — NOT the fix story. No B-numbers, round numbers, or test-name narratives in the entry body (the session note carries those); no new per-round `### Round-NNN gotchas` sections — file new entries under the matching topical section. If an entry needs more than ~5 lines, the invariant probably isn't isolated yet — distill it.
- **THE ARCHIVE IS NOW MOST OF WHAT THIS FILE USED TO SAY (2026-07-26 trim, 425 KB → ~91 KB, owner's context-engineering directive: "less is more").** Every per-diagnostic and per-walker section — Type system / Checker / Overload resolution / Module detection+resolution / Binder / Scanner-Parser / Emitter / Transformer / Lib-suggestion / all the per-TS-code sections — moved WHOLESALE to `docs/history/CLAUDE-GOTCHAS-ARCHIVE.md` (656 KB). Nothing was deleted. What stayed here: cross-cutting architecture of LIVE subsystems, process/build traps, measured negative knowledge, test conventions, Kotlin idioms, and the mission/protocol. **SO: when you touch a dedicated walker, a specific TS code, a corpus-test regression, or a frozen subsystem, GREP THE ARCHIVE FIRST** (`command grep -n '<walker|TSxxxx|test name|B-number>' docs/history/CLAUDE-GOTCHAS-ARCHIVE.md`) — the knowledge is there, it is just no longer loaded into every session. Progressive disclosure is the design; treat a missing entry here as "look in the archive", never as "no such constraint exists".
- **The archive (2026-07-06, Phase 17 residency rule)**: ~250 corpus-era entries — corpus-unique pin/dedicated-walker docs (B-numbers), per-test parser-recovery cascades, emitter/baseline byte-format minutiae, checkJs one-offs — live in `docs/history/CLAUDE-GOTCHAS-ARCHIVE.md`. Their protection is the always-green corpus suite (a 2-minute hard gate), not this file. **Grep the archive (by walker name / B-number / corpus test name / TS-code) BEFORE modifying or deleting a dedicated walker, investigating a corpus-test regression, or working in a frozen subsystem (parser error-recovery, comment emit, baseline formatting, decorator metadata).** Residency here is reserved for: cross-cutting architecture of LIVE subsystems, process/build traps, and measured negative knowledge ("X was tried and regressed") — per-test/per-walker detail goes straight to the archive.

## Known gotchas


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xemantic/xemantic-typescript-compiler](https://github.com/xemantic/xemantic-typescript-compiler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
