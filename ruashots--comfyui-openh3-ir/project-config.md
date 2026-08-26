---
trigger: always_on
description: **This document is for changing the node pack.** Read it before changing anything. It is the set of
---

# Working on the OpenH3-IR node pack

**This document is for changing the node pack.** Read it before changing anything. It is the set of
rules that are not preferences, a map of which file owns what, the ComfyUI frontend behaviour that
was measured rather than assumed, and how to prove a change is really live in a running ComfyUI.
There is no install path here, on purpose: that is README.md beside this file.

The compiler is the other half and it is another repository, `open-h3-ir`. This pack depends on the
published package and carries no copy of it: a compiler bug is fixed and released there, and nothing
here changes for it. Its own maintainer document is `AGENTS.md` in that repository, and the rules
about writing briefs, prompts, validators and evaluation all live there.

## The two things to hold your work against

```bash
pytest tests                                   # from the repository root, and the path matters
.venv/bin/python research/contract_falsification.py
```

**`pytest tests`, never a bare `pytest`.** This repository's root IS the pack, so it holds an
`__init__.py`, and ComfyUI Manager clones it into `custom_nodes` under a name with hyphens in it.
pytest turns any directory holding an `__init__.py` into a Package and imports that file before
running anything below it, and it names the module by walking up while each folder is a Python
identifier -- which `ComfyUI-OpenH3-IR` is not. So it imports as a bare `__init__` with no parent
package and the first relative import raises, in every test. `tests/pytest.ini` puts the rootdir
inside `tests`, which only takes effect when the path handed to pytest is inside it, and
`conftest.py` at the root turns the other invocation into one sentence. Measured on pytest 9.1.1;
neither `--import-mode=importlib` nor `consider_namespace_packages` nor `--ignore` avoids it.

The test count is not pinned here, because it moves every time anyone adds a test and a stale number
reads as a regression.

**The tests need `open-h3-ir` installed**, at or above the release `requirements.txt` names. Some of
them read the contract that package publishes and compare it to the copies this pack ships, which is
the whole point: it holds this pack against the released compiler rather than against a working tree
nobody has.

## What the README leads with

The owner's words, 2026-08-23, and they are the pitch rather than a description:

> no more copy pasting prompts from a chat with an llm, no more explaining in what slot a resource
> is so the llm can name it right, no more explaining what a resource contains, OpenH3-IR takes
> care of that directly in comfy

Every clause of that is a thing this pack actually removes, so none of it is a claim anybody has to
soften:

- **The copy and paste.** The compiler writes the brief. Nobody carries text back from a chat window.
- **Naming a slot so a model gets the label right.** Labels are computed in the compiler's `plan.py`,
  never asked for. That is rule one of the codebase and it is why the labels are always right.
- **Explaining what a file holds.** The reference pictures and clips are read through the language
  model, so a person says `@carguy` and nothing else.
- **Leaving ComfyUI to do any of it.** The compiler runs in the same Python ComfyUI runs.

Written down here because the README gets its final pass after the Main node lands, and a pitch the
owner said once in a message is the sort of thing that gets lost between now and then.

## Where things are

The repository root is the pack. That is not a layout preference: ComfyUI Manager clones a repository
straight into `custom_nodes` and imports the cloned directory's top level, so anything one folder
deeper produces zero nodes and no error anybody can act on.

The ComfyUI pack in the repository root is eight Python files plus a `web/` folder of five JS files. Exactly one of them names `h3ir`, and every one of those imports is inside a function. Two files are generated; both say so at the top:

| file | what it owns |
|---|---|
| `h3ir_client.py` | the service protocol, the option lists, the report, and the refusal sentences BOTH compile paths use. No ComfyUI, no torch, no third-party packages, no `h3ir`. |
| `compiler.py` | running the compiler in this Python: is it installed, what does it publish, which language model writes, does that model see, and the compile itself. **The only module that imports `h3ir`.** |
| `media.py` | tensors and mappings to files on disk, content-addressed. No ComfyUI at module scope. |
| `nodes.py` | the four node schemas -- Main, Media, Setup and the optional Director -- the model loaders and the socket-to-file mapping. This is the only file that needs a canvas. |
| `contract.py` | the snapshot of the contract this pack was built against, `Half` (which compiler this graph uses and what to call it), and the decision about what a difference costs: what stops a queue, what is a line in the report. |
| `contract.json` | GENERATED. `h3ir contract` wrote it. The snapshot the file above reads. |
| `web/contract.data.js` | GENERATED. `h3ir contract --js` wrote it. The seven profiles, the camera table and the cap, for a panel that has to draw with nothing running. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ruashots/ComfyUI-OpenH3-IR](https://github.com/ruashots/ComfyUI-OpenH3-IR) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
