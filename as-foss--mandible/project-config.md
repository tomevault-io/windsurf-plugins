---
trigger: always_on
description: Read this before touching code. It is short on purpose. Every entry exists
---

# AGENTS.md — working agreements for AI agents in this repo

Read this before touching code. It is short on purpose. Every entry exists
because something actually went wrong, and it says which failure it prevents so
a future reader can judge whether it still applies.

**Precedence:** `spec.md` is the design authority — what to build and why. This
file is the operational authority — how to work here without repeating known
mistakes. `CONTRIBUTING.md` is for humans. If this file and `spec.md` disagree
about design, `spec.md` wins and you should fix this file.

---

## 1. The invariant that defines the project

> **No per-tool logic, ever.** No `if tool == "docker"`, no tool-name-keyed
> special case in any extraction tier, no per-tool patch file vendored into this
> repo.

Tool-specific knowledge lives in exactly one place: user-local override files
under `~/.config/mandible/overrides/`, which are never committed here. (Spec
revision 3 deleted the vendored catalog that used to be the second place — a
per-tool catalog is per-tool knowledge relocated into data, and it cannot stay
current with the tool actually installed. Parsing is keyed by *framework* now:
see spec §7 Tier A′ and `mandible-extract/src/help_text/profile.rs`, where
adding a framework is one `match` arm plus one fingerprint.)

If a tool renders badly, the fix is a better general parser, a new general tier,
or an honest low-confidence badge in the UI. It is never a special case. This is
the entire reason the tiered architecture exists; one exception starts the
erosion that the architecture was built to prevent.

---

## 2. Architectural invariants

Breaking any of these produces a bug that tests will not catch.

| Invariant | Where | Failure it prevents |
|---|---|---|
| `Text::sanitize` (or `sanitize_markdown`) is the **only** way untrusted text enters the IR | `mandible-core/src/text.rs` | Control chars and markup reaching a `ratatui::Span`, which corrupts pane borders. Two widget-level fixes for this failed before the boundary fix worked. |
| Widgets may **assume** `Text` is clean | `mandible-tui` | Re-implementing defenses in each of the three consumers (tree, detail, clipboard), inconsistently |
| `std::process` appears **only** in `mandible-extract/src/exec/` | enforced by `tests/no_process_outside_exec.rs` | Unaudited subprocess spawning; §6 of the spec becomes unenforceable |
| Provenance is **per field**, never per tree | `mandible-core/src/provenance.rs` | A trust badge that lies after a multi-tier merge — worse than no badge |
| Extraction is **node-scoped** (`extract_node`), never whole-tree | `mandible-extract/src/tier.rs` | Eager extraction: 232 subprocesses and 10.5s for `docker`. Do not reintroduce a whole-tree `extract()`. |
| **One node = exactly one tree row.** No wrapping in the tree pane | `mandible-tui/src/render/tree_pane.rs` | Row index ↔ node stops being a bijection, breaking selection, scrolling, mouse hit-testing, and filtering all at once |
| Truncate by **display width** (`unicode-width`), never `char` or byte count | `mandible-tui` | CJK/emoji overflow the border by one cell per wide character |
| Never slice a `&str` derived from tool output at a raw byte offset (`&s[..n]`) | any tier that parses `--help`/similar text | Panics if the offset isn't a UTF-8 char boundary. Shipped as a real crash (`help_text::sections`, found by the coverage harness's first real run, not a synthetic test): a box-drawing glyph early in one real tool's output landed byte 6 mid-character. Use `s.as_bytes().get(..n)` (bounds-checked, no boundary concept for raw bytes) for ASCII-prefix checks, or `s.get(..n)` (returns `None` instead of panicking) generally. |
| A bare-word block becomes subcommands only under a *recognized* heading (or a chain started by one) plus a name-shape check — never from layout alone | `mandible-extract/src/help_text/sections.rs` | [M-10]: Tier B fabricated 39-65 phantom subcommands per tool from wrapped description continuation lines and `--format=`-style enum value lists. Fabricated structure is worse than missing structure — a user can't tell it's wrong. The coverage harness's structure-sanity column (spec §13.1) is the regression net: `%described` alone stayed at 100% while this was happening. |
| Programs whose purpose is to kill processes (`kill`, `pkill`, `killall`, `fuser`, `reboot`, …) are **never executed**, under any argv | `mandible-extract/src/exec/spawn.rs` (`NEVER_PROBE`), enforced in `run_inert` | `mandible pkill` froze a user's machine into a reset. `--help` being harmless on one build is not enough: rule 2 permits `<tool> <word> --help`, and for these the first positional is a *target* — `killall foo --help` kills everything named `foo`. This is a **safety** list, not the per-tool parsing knowledge §1 forbids: it is closed, and keyed on what a program *does*, not on how its output is formatted. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AS-FOSS/mandible](https://github.com/AS-FOSS/mandible) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-10 -->
