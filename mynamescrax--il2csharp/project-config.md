---
trigger: always_on
description: `il2csharp` is an IL2CPP → C# decompiler: `global-metadata.dat` +
---

# AGENTS.md — new-session tutorial

`il2csharp` is an IL2CPP → C# decompiler: `global-metadata.dat` +
`GameAssembly.dll` in, a C# tree out, every body lifted from native x64.
This file orients a new session. Normative rules live in `CLAUDE.md`;
start there, then `docs/todo.md` (Current work section).

## First reads (in order)

1. `CLAUDE.md` — architecture, mandatory guardrails, validation gates.
2. `docs/todo.md` — current work (top section is live), what's done,
   what's deferred with evidence.
3. `docs/handoff-2026-09-23.md` — 2026-09-23 stop record (historical);
   the 2026-09-22 failure-class inventory (the former root `TODONOW.md`)
   is now an appendix of `docs/todo.md`. Live work is `docs/todo.md`;
   the pre-reorg `nowtodo.md` no longer exists.
4. `docs/README.md` — map of every doc and `§NN` cross-reference.
5. `docs/construct-mapping.md` — what native constructs recover as what
   C#, plus the detailed internals/validation notes.

## Code map

- `il2csharp.py` — thin CLI launcher (CRLF **with** BOM).
- `il2cpp/` — the package (all CRLF, no BOM). Public names re-exported
  from `il2cpp/__init__.py`:
  - `metadata.py`, `binary.py` — `global-metadata.dat`, PE/ELF frontends.
  - `runtime/` — `Il2Cpp`: registrations, usage slots, EH4 (`core.py`
    holds the class; `registration`/`types`/`fields`/`eh` mixins).
  - `lifter/` — symbolic x64 over iced-x86 (`state`, `values`, `insn`,
    `calls`, `render` mixins; `aggregates.py` = stack-tile/struct
    provenance).
  - `expr.py` — `Expr` values (text + il2cpp type tuple + kind).
  - `dec/` — CFG + structured decompiler: `build` (blocks, site scans),
    `analyze` (dry/real exec, merges, phi copies), `structure`
    (the ~25-stage statement pipeline — pass order matters, read it
    before reordering), `flow`/`highlevel` (sugar passes incl.
    `_switch_synth`, `_hash_string_switch`), `textpass` (final
    `*(E+N)` → `((byte*)E+N)[0]` rendering), `emit`, others.
  - `emitter.py`, `headers.py`, `cli.py`, `arm64.py` scaffold.
- `tests/` — portable unit tests (LF) + game goldens
  (`test_game_goldens.py`, `tests/goldens_review84.json` — frozen).
- `tools/` — `inspect_methods.py` (`--mi` exact MethodDef rows),
  `corpus_common.py` (fixture loader). Add `tools/` to `PYTHONPATH`.
- `work/` — scratch runners (`work/lib/`: `sweep_audit.py`,
  `tree_brace_audit.py`, `ts_gate.py`; routing in `work/README.md`).
- `testgame/` — local, git-ignored licensed fixture (never tracked or
  redistributed; see `docs/public_release.md`).
- `final_out/` — last promoted tree (r10, 2026-09-28). Read-only reference.
  Never edit, never rebuild into.
- `validation_reports/` — frozen gate evidence. Don't touch.
- Temp scratch: `C:\Users\crax\AppData\Local\Temp\opencode` (outside repo).

## Session setup (PowerShell)

```powershell
$env:PYTHONHASHSEED = '0'
$env:IL2CSHARP_METADATA = (Resolve-Path 'testgame/ShiftAtMidnight_Data/il2cpp_data/Metadata/global-metadata.dat').Path
$env:IL2CSHARP_BINARY = (Resolve-Path 'testgame/GameAssembly.dll').Path
$env:PYTHONPATH = '<repo>;tools'
```

## Common tasks

**Lift one method** (the fast loop — no file I/O):
`tools/inspect_methods.py --metadata ... --binary ... --mi 25293`.
Goldens key on MethodDef row (`mi`), never VA (shared bodies alias).

**Portable tests:** `python -m pytest -q -m "not game" --disable-warnings`
(943 pass). **Full suite:** `python -m pytest -q` (~7 min, needs fixture
env above; suite is fully green — any failure is yours).

**Rebuild a tree:** `python il2csharp.py testgame -o <name_out1> [--only
Assembly-CSharp] [--workers N]`. `--workers 0` = half the cores (max 8);
the tree is byte-identical, only the schedule changes; `--max-methods`
forces 1. Name output `*_out1/` (git-ignored). Gate with
`python work/lib/tree_brace_audit.py <dir>` (must print 0 unbalanced).

**Land a source fix:** repro first (probe script in temp dir, never in
repo), binary-safe patch (below), compileall, portable suite, targeted
`--mi` re-lifts, full-suite recount (failure SET must be byte-identical
to the triaged list), docs, commit, push. Decline-by-default: every
unproven shape keeps today's spelling; raw is always honest.

## Iron contracts (violations cause phantom diffs / flaky output)

- `il2cpp/*.py` are CRLF no-BOM: edit via binary read/write scripts,
  assert `d.count(b'\r\n') == d.count(b'\n')` after every touch.
  Read-text/write-text, `sed -i`, and heredocs silently rewrite to LF.
  `tests/*.py` and `*.md` are LF.
- Never open the target for writing until assertions pass (build in
  memory, assert, write once); never re-derive backups from live files.
- PowerShell 5.1: no `head`/`tail`/`grep`/`rm`/`||`/`&&`; no `>` redirect
  (writes UTF-16); quote paths with spaces. `PYTHONHASHSEED=0` always —
  set iteration order leaks into temp names otherwise.
- GitHub is the history authority for the post-scrub history. The
  2026-09-28 scrub removed the licensed fixture and every full-body /
  disassembly artifact from all of it and pruned LFS, so every pre-scrub
  commit hash changed; the intended local backup bundle is no longer
  present (see `docs/public_release.md`), so hashes in older docs no
  longer resolve anywhere. Never track or publish game data or

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mynamescrax/il2csharp](https://github.com/mynamescrax/il2csharp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
