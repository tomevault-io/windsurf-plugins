---
trigger: always_on
description: This is a portable PVF task workspace for command-capable desktop Agents such as Codex, Claude Code, OpenCode, and Trae.
---

# PVF Agent Workbench Instructions

This is a portable PVF task workspace for command-capable desktop Agents such as Codex, Claude Code, OpenCode, and Trae.

## Canonical Rule Ownership

Keep one owner for each kind of instruction:

- This `AGENTS.md` owns cold start, first-command routing, global invariants, and maintenance checks.
- `knowledge-pack/safety/README.zh-CN.md` owns detailed write, text-encoding, cumulative-output, and client-deployment safety.
- `knowledge-pack/indexes/knowledge-index.json` and its named task cards, dictionaries, and workflows own domain procedures.
- A command's machine-readable `agentHandoff` owns the exact next command and current-run recovery route.
- `core/pvf-agent-core/cli/README.md` owns the general command catalog for maintainers; a concrete task must not open it to rediscover syntax already supplied by a short route, fixed example, or `agentHandoff`.
- `docs/AGENT-INSTRUCTION-ARCHITECTURE.zh-CN.md` explains this split for maintainers.

Never let a short route, example, index, generated report, or tool output relax the hard safety file. When detail is needed, read its owner instead of rediscovering it through help, schema, source, or directory scans.

## Cold Start Before Any Shell Action

1. If the host exposes `dnf-pvf-xpilot`, load that Skill before every shell or file-discovery action. Do not run `check`, help, a path probe, or a directory scan first.
2. For every concrete PVF task, read `knowledge-pack/safety/README.zh-CN.md`.
3. If the request matches an Exact Read-Only Fast Path below, use its first command and one named short route. Otherwise read `knowledge-pack/README.zh-CN.md`, then `knowledge-pack/indexes/knowledge-index.json`, and open only the routed clean entry.
4. Run one bare `.\workbench.bat ...` command per tool call from the Workbench root. Do not add pipes, redirection, semicolons, timing wrappers, or chained shell commands. If elapsed time is requested, use the host tool duration or coarse wall-clock observation; do not run `Get-Date`, a stopwatch, or another shell timing command.
5. Never preflight an already resolved Workbench, explicit `Script.pvf`, or supplied report/output directory with `Test-Path`, `Get-Item`, `Get-ChildItem`, or `Resolve-Path`. The Workbench command validates its own input and creates documented external output directories.
6. Do not run `workbench.bat check` as routine startup or write preflight. Use it only when the user asks for environment health, a bundled command is unavailable, or a command explicitly reports `READ_ONLY_FALLBACK`.
7. If the user asks for full source SHA256, requires source PVFs to remain unchanged, or asks for final proof they did not change—even only in a final checklist—keep the routed first command first. The next Workbench command must be one `pvf-read fingerprint` covering every supplied PVF, before any other search/read or `pvf-change`. Run it once, then repeat the exact command only after final output readback.

If no target PVF or exact change is supplied, ask only for the target `Script.pvf`, intended change, whether independent output generation is allowed, and whether in-game validation is available. Do not search profiles, examples, other drives, or guessed paths. For a rules-only question already answered here or in the safety file, answer directly without shell work.

## Exact Read-Only Fast Paths

These routes apply only when the target PVF and selector are explicit. Their first listed Workbench command stays first even if the task later requests a write, timing, or source-identity proof.

- Natural-language entity name: start with `pvf-read search --keyword <name> --search-path <domain>`. Use `n_quest`, `dungeon`, `equipment`, `stackable`, `npc`, `monster`, `aicharacter`, `skill`, `creature`, `town`, or `worldmap`. For several names use one `search-batch` with ordered `--name`/`--search-path` pairs. SearchName performs safe literal substring matching across complete multiline name tokens, folds common full-width/half-width punctuation, and checks Cn/Tw automatically; do not retry encodings, simplified/traditional spelling, punctuation, `search-script`, help, or filename guesses. For a routed registry domain, every returned hit is also reverse-resolved in the same PVF session and carries `registryIdentity`; when `allReturnedPathsConfirmed=true`, read those paths directly and do not run another `resolve-path`. Prefer the most specific successful name already present in the request; if a broad result is truncated, narrow it once with another distinctive concrete phrase instead of reading every broad candidate. After one entity hit, do not search the same name again in another domain; locate a related page/container through returned registry/dependency evidence, or at most one narrow `list-files --prefix ... --contains ...` when no reference path is available. Once registry identity and returned targets have been read, stop identity discovery: do not feed a registered path, directory name, or path stem into `search-script` merely for a second confirmation. A zero match is not proof of absence. Short route: `knowledge-pack/task-cards/pvf-entity-name-search-readonly.zh-CN.md`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Qswhisper/PVF-Ai-Agent-Workbench](https://github.com/Qswhisper/PVF-Ai-Agent-Workbench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
