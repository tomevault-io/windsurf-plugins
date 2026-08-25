---
trigger: always_on
description: Read this before changing anything in this repository. It covers where to look, what has to change *together*,
---

# Working on GMP as an agent

Read this before changing anything in this repository. It covers where to look, what has to change *together*,
and the mistakes that are easy to make here specifically.

Human contributors: this is also the shortest description of the repo's moving parts.

---

## 1. Where to look

**Start with `wiki/`, not with the source.** It is 33 pages of concept and API reference, written to be read
before the code, and `wiki/Home.md` indexes all of it. Two pages answer most "how is this supposed to work"
questions: `Dispatch layers` (who hears a send) and `Build switches` (every macro and what it changes).

| Path | What it is |
|---|---|
| `Plugins/GMP/Source/GMP/GMP/` | Public headers. `GMPHub.h` is the send/listen entry, everything templated lives here |
| `Plugins/GMP/Source/GMP/Private/` | Implementation, plus `GMPTests.cpp` (83 automation tests) |
| `Plugins/GMP/Source/GMP/Shared/` | One header per script backend: slua, UnLua, Puerts, AngelScript, C# |
| `Plugins/GMP/Source/GMPEditor/` | K2Nodes, codegen, editor tooling — the largest module by file count |
| `wiki/` | **Source** for the GitHub wiki; CI mirrors it. Never edit the wiki online, it gets overwritten |
| `docs/` | GitHub Pages. `*.html` is generated — edit the `.md` and rerun the generator |
| `tools/` | `gen-diagrams.py`, `build-docs.py`, `build-wiki.py` |
| `doc/` | Untracked scratch (`.gitignore`d). Working notes, not part of the repo |

Searching: message APIs come in families, so grep the family name rather than one function —
`ListenObjectMessage` has by-key, by-index, direct and script variants that must stay consistent with each other.
`wiki/ListenMessage family.md` and `wiki/NotifyMessage family.md` enumerate them.

---

## 2. What has to change together

This is the part that gets missed. Each row is "if you touch the left, you must also touch the right".

### A message API

| Changed | Must follow |
|---|---|
| A send/listen form in `GMPHub.h` | The **five** script backends in `Shared/` — they are deliberately symmetric. A form that exists in C++ but not in Lua is a bug report waiting to happen |
| Anything script-facing | Blueprint too: `GMPBPLib.h/.cpp`, and often a K2Node in `GMPEditor/` |
| AngelScript specifically | Its callbacks are typed, so the generated declarations in `GMPEditor/.../GMPAngelScriptCodeGen.cpp` must match what the runtime registers — name, signature and parameter order, exactly |
| Any of the above | A test in `GMPTests.cpp`, and the matching `wiki/` page |

Script backends share one entry wherever possible: collection rows all go through `GMPListenScriptRows`, and each
backend supplies only the bridge from the argument addresses to its own callable. **Follow that pattern** rather
than reimplementing dispatch per backend.

### Docs and figures

| Changed | Must follow |
|---|---|
| `README.md` / `README_CN.md` | `python tools/build-docs.py` — `docs/index.html` and `index-cn.html` are generated from them |
| `docs/article-*.md` | Same generator; each article is a `subpage()` call at the bottom of `build-docs.py` |
| A figure | `tools/gen-diagrams.py` — **figures are code, never hand-edit `docs/img/`**. Rerun the script |
| Adding a figure | Write an `fNN()` function, register it in the list at the bottom, and reference it from the docs |
| `wiki/*.md` | Nothing manual: `sync-wiki.yml` mirrors it on push. Verify locally with `python tools/build-wiki.py --target <dir>`, which also reports broken `[[links]]` |
| A statement in `wiki/` about behaviour | Check the other pages that describe the same behaviour. Encoding rules in particular are documented in several places and drift silently |

Figures are **lossless webp**, static and animated alike — do not add png or gif. The generator quantises before
encoding, which is what keeps them small; skipping that step makes files *larger* than the png they replace.

### Behaviour that is configuration-dependent

Anything guarded by a `GMP_WITH_*` macro has at least two behaviours. `GMP_WITH_DYNAMIC_CALL_CHECK` (editor and
development) and `GMP_WITH_MSG_HOLDER` are the two you will meet most. If a change only makes sense under one of
them, say so in the code and make sure the other path still compiles — a monolithic shipping build resolves
stores at compile time and takes entirely different branches.

---

## 3. Mistakes that are specific to this repo

**`ensure` is diagnostics, not control flow.** It does not stop execution in shipping. A constructor that fails
its `ensureMsgf` must leave the object in a state where every subsequent call is a safe no-op, or the ensure just
becomes a null dereference one line later.

**`checkSlow` only exists in debug.** Any check whose failure would corrupt memory — a type mismatch before a
`reinterpret_cast`, for instance — has to be `ensure` at minimum. `checkSlow` for that is the same as no check in
every build anyone ships or profiles.

**Skip `IsEditorOnlyProperty()` when walking reflected members positionally.** A `WITH_EDITORONLY_DATA` member
sits in the middle of the member sequence in editor and vanishes in shipping, so the same index reads a different
field per configuration. This reproduces only in a packaged build.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wangjieest/GenericMessagePlugin](https://github.com/wangjieest/GenericMessagePlugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
