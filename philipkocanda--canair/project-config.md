---
trigger: always_on
description: For reference, the WiCAN firmware is checked out in the `wican-fw/` directory (gitignored; pull if you need to reference the latest version).
---

# Hyundai Ioniq 2017 — CAN Reverse Engineering

For reference, the WiCAN firmware is checked out in the `wican-fw/` directory (gitignored; pull if you need to reference the latest version).

## Running the CLI — ALWAYS use `uv run canair` from the repo root

**Every `canair` invocation in this repo MUST be run as `uv run canair …` from the project root (`/Users/philip/projects/canair`).** Do NOT call a bare `canair` (a globally installed `uv tool install .` copy). Running bare `canair`:

- may execute **stale code** from a previous install instead of the current working tree, and
- may resolve a **different vehicle profile** (wrong `--profiles-dir` / `default_profile`) than the repo-bundled `profiles/ioniq-2017/`.

Running `uv run canair …` from the repo root guarantees the latest repo code and the expected in-repo profile. Throughout this file and the skills, wherever an example is written as `canair …`, execute it as `uv run canair …` (use the `workdir` of the repo root; never `cd`-hop away from it).

## Prefer canair's built-in tooling — do not hand-roll

For **all** querying, capturing, analysis, and reverse-engineering, use the dedicated `canair` subcommands documented below rather than raw sockets, ad-hoc scripts, or hand-editing YAML:

- **Query/read the device** with `canair query` / `scan` / `discover` / `io` / `routines` (use `raw` only as a last resort).
- **Analyze historical data** with `canair captures`, `canair decode`, `canair correlate`, `canair hunt`, `canair investigate`, `canair coverage`, `canair research`.
- **Edit definitions** with `canair pids` (surgical, validated) — not by hand-editing `ecus/` YAML.
- **Always pass `--save`** (with `--label`, `--state`, `--notes`) when reading from the device so every payload is recorded to `captures/`. See **Key Files → captures** below for the full flag set and examples. `--save` works with `query`/`scan`/`raw`/`discover`, positional query/raw steps, and `--monitor`. Saves are **journaled** (a write-ahead log under `captures/.journal/`) and reconciled into the dated capture file on exit, so a killed/crashed/disconnected session is never lost — recover leftovers with **`canair captures uds --recover`** (`--discard` to drop them). In `--monitor` press **`s`** to set/edit the label/state/notes of the **current** session live (the save modal names which segment it labels), or **`n`** to **close the current segment and start a fresh one** (each segment reconciles to its own capture file, so one run yields several labelled sessions); an active `--save` recording shows a blinking **`● REC`** in the status line. The **vehicle state is auto-suggested** from decoded PID values via the profile's `states.yaml` (see `canair validate states`). Repeated non-`--save` on-demand saves only write payloads seen **since the last save** (no duplicates).

## Boy Scout rule — leave it better than you found it

When you notice something wrong while working — a bug, a typo, a stale comment, a small inconsistency, a missing validation, dead code — **be inclined to just fix it** as part of your change. Prefer improving the tree over walking past a defect.

The exception: if the fix is **too much work** (a large refactor, a risky/behavior-changing edit, anything touching sensitive/irreversible operations, or something clearly outside the current task's scope), **ask first** before diving in. Surface what you spotted, describe the fix, and let the user decide. Don't silently balloon a small task into a large one.

## Keep docs & README current — every user-facing change

Docs are part of the change, not an afterthought — **any** change that adds, removes, or alters a user-facing capability (a new/renamed subcommand, a changed/added/removed flag, a shifted default, new setup/config steps, a changed workflow, a new profile field) MUST update the user-facing docs in the same change. Stale docs mislead as badly as stale code. If nothing user-facing changed, confirm that rather than assume.

**Respect the README ↔ `docs/` split:**

- **`README.md` stays compact and high-level** — the landing/gateway page: intro, the connection diagram, the command *map* (one crisp line per subcommand), a short quick-start, the bring-your-own-car *arc*, bundled-profile highlights, license, warning. **Detail does not belong here**; every section links *into* `docs/`. It was deliberately trimmed (311→143 lines) — keep it lean, don't re-expand it into a manual.
- **`docs/` carries the detail** — task-first, for **new-car users** and **PID/profile contributors**: `getting-started/`, the `bring-your-own-car/` journey, `concepts/`, `reference/`. New walkthroughs, worked examples, and per-command flag detail go here. `docs/` is the human-facing rendering of the same knowledge in this file and the skills — it should *reference* them, not duplicate; where a fact is authoritative elsewhere (`config.example.yaml`, `--help`, `canlib/schema/`), point at / derive from it so it can't drift.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [philipkocanda/canair](https://github.com/philipkocanda/canair) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
