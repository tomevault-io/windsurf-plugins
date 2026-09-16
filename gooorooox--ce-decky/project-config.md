---
trigger: always_on
description: The compact entry point for coding agents. Everything else this project holds is in `docs/`, and **Authority and reading routes** below says which document owns what and which one wins.
---

# AGENTS.md: CE Decky operating contract

The compact entry point for coding agents. Everything else this project holds is in `docs/`, and **Authority and reading routes** below says which document owns what and which one wins.

Current development version: **0.9.27 — 2026-09-14**

## Start here

On every clean run:

1. Read this file completely, in bounded slices. It ends with **Release discipline**: if that heading is not in what you just read, the read was truncated and the rest of this contract is not optional. A route that truncated once truncates again, so change the route rather than repeating it.
2. Read `docs/FIELD_NOTES.md` **Still worth checking**, which is the whole of what this project knows it has not confirmed. It is empty when nothing is owed, and that is an answer.
3. Before feature or bug-fix work, read only the newest `CHANGELOG.md` entry.
4. Classify the change with the routing table and open only the required contracts and source files.
5. Before writing any command that reads repository or device state, check **Tracked helpers**. One already answers most such questions, and the ones it does not are named in `docs/DEVELOPMENT.md`.

## Mission and implementation policy

Build CE Decky as a Decky Loader plugin that manages Cheat Engine tables and, after explicit target validation, runs either the exact plugin-managed official Windows Cheat Engine installation or a user-imported Windows installation in the selected game's Proton environment. Decky/QAM is the primary control surface.

**Full Game Mode autonomy is a release requirement.** A normal user must be able to complete setup, game and table selection, search/download/import, exact-table review and consent, table switching, process/startup configuration, launch, live per-record control, recovery, and routine updates with a controller in Steam Game Mode. Desktop Mode, a terminal, SSH, manual filesystem edits, and developer harnesses are evidence tools only. Manual CE/table import is a controller-accessible fallback, not a prerequisite.

For material Decky, Steam, Proton, Cheat Engine, archive, provider, or launch integration decisions:

1. Treat remembered integration knowledge as provisional; inspect current upstream source first.
2. Prefer the official project/template. Corroborate undocumented conventions with maintained real plugins.
3. Start with `docs/DESIGN.md` section 4 and `docs/FIELD_NOTES.md` **Upstream references**, which pins the commit each reused behavior was read from.
4. Inspect the pinned source and license before adapting it. Record repository, commit/tag, path, and whether code was copied, adapted, or used only as evidence.
5. Retain attribution/license text, add a regression for relied-on behavior, and preserve CE Decky's stricter trust boundaries.

Upstream source is precedent, not production code. Do not discard a unique upstream identity, a license finding, an observed failure, or a target assumption without first preserving it in a durable contract.

## Authority and reading routes

If documents disagree, use this order: `AGENTS.md` for workflow/invariants, `docs/DESIGN.md` for product behavior, `docs/ARCHITECTURE.md` for boundaries, `docs/SECURITY.md` for trust controls, and `docs/FIELD_NOTES.md` for what a device established and what it has not.

| Change | Read before editing | Update if the contract changes |
|---|---|---|
| Documentation/repository layout | affected files | affected document |
| Product behavior/workflow | relevant `docs/DESIGN.md`; source/tests | `docs/DESIGN.md` and tests |
| Component/data/storage boundary | relevant `docs/ARCHITECTURE.md` and `docs/DESIGN.md` | `docs/ARCHITECTURE.md`; `docs/SECURITY.md` if trust changes |
| Artifact, network, CE runtime, or Steam-state mutation | relevant design/architecture plus `docs/SECURITY.md` | contracts and regressions |
| Decky, Steam, Proton, or CE target behavior | affected contract; `docs/FIELD_NOTES.md` for what a device already established | `docs/FIELD_NOTES.md` for a durable conclusion, and only for one |
| External-code reuse/replacement | reuse sources above and `docs/FIELD_NOTES.md` **Upstream references** | attribution/notices, regression, design reference if selection changes |
| Build, dependencies, CI, package, release | `docs/DEVELOPMENT.md`, manifests, workflows, scripts | development/release docs and changelog as applicable |
| Decky Store/publication | `docs/DEVELOPMENT.md` **The artifact the Store builds**, and `docs/FIELD_NOTES.md` **Upstream references** for the build contract that was read rather than remembered | the procedure, and `docs/FIELD_NOTES.md` if what upstream does changed |

Read a whole contract before changing it.

## Non-negotiable product invariants

- No anti-cheat bypass, disabling, evasion, or stealth.
- Do not bundle Cheat Engine binaries/source or modify the selected CE installation. A private runtime copy is local, plugin-owned, and never distributed.
- `.CT` import is not execution consent. Lua, Auto Assembler, and embedded payloads are executable content.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [goooroooX/CE-Decky](https://github.com/goooroooX/CE-Decky) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
