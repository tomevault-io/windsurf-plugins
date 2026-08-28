---
trigger: always_on
description: This repo is a generator shell, not a collection of plugins. Every generated
---

# JUCE-MCP: prompt-driven JUCE plugin generator

This repo is a generator shell, not a collection of plugins. Every generated
plugin project (e.g. a folder you or a prior session created) is a local
build artifact — gitignored, never committed. The repo itself only carries
the scaffolding template, the build toolkit, and instructions.

## Submodules

`JUCE-Plugin-Starter/` and `juce-agent-toolkit/` are required and must be
initialized before generating anything:
```bash
git submodule update --init JUCE-Plugin-Starter juce-agent-toolkit
```
`ai-enhanced-audio-book/`, `juce-docs-mcp-server/`, and `JuceMCP-for-agents/`
are optional submodules — initialize individually only if needed (see
README.md).

## Generating a plugin

Three paths, all ending the same way: a new `<plugin-name>/` directory
scaffolded from `JUCE-Plugin-Starter/` as a **sibling of this repo** (one
level up — see `--destination-parent ..` below), built with CMake/Ninja, and
tested with Catch2. Scaffolding outside this working tree is deliberate: it
means a generated project never needs its own `.gitignore` entry here. Works
the same way on macOS, Windows, and Linux — see "Cross-platform CMake
snippets" below for the canonical patterns any neural-FX plugin needs to
actually build on all three.

Use `uv` for any Python tooling a generated plugin or this workflow needs
(`uv venv`, `uv pip install`, `uv tool install`) — not plain `pip`/
`python -m venv`. See README.md's "One-time setup: libtorch" for the exact
commands.

**From a prompt.** The user describes the DSP/behavior they want. Scaffold via:
```bash
python3 juce-agent-toolkit/shared/scripts/create_project.py "Plugin Name" \
  --starter ./JUCE-Plugin-Starter \
  --destination-parent .. \
  --developer-name "<name>"
```
**Confirmed on macOS**: the system `/usr/bin/python3` can be old enough
(3.9) that `create_project.py`'s own `from find_starter_repo import locate`
fails with `TypeError: unsupported operand type(s) for |: 'type' and
'NoneType'` — `find_starter_repo.py` uses `Path | None` union-type syntax,
which needs Python 3.10+. Confirm with `python3 --version` first; if it's
older, run the script with a newer interpreter instead of trying to fix the
toolkit script (it's a submodule): `uv run --python
$(uv python find 3.12 2>/dev/null || echo /opt/homebrew/bin/python3)
juce-agent-toolkit/shared/scripts/create_project.py ...`, or any other
3.10+ interpreter on the machine.

then implement the processor/editor, build (`cmake -B build -G Ninja` inside
the new project dir, then `cmake --build build --target <Name>_Standalone`),
and write Catch2 tests that actually push real audio through `processBlock`
and check for NaN/Inf and no exceptions — not just construction. For DSP
whose whole point is a specific frequency/amplitude behavior (an EQ, a
compressor, a filter), a passing NaN-check test doesn't confirm the plugin
does what it claims — also assert the actual behavior, e.g. feed a sine at
a target frequency and check steady-state RMS moved by the expected amount,
and feed a sine far from any targeted band and confirm it *didn't* move
(confirmed on Torch Parametric EQ: a 3-band parametric EQ whose coefficient
design is the standard RBJ Audio EQ Cookbook math in C++, but whose actual
recursive IIR filtering runs through a TorchScript-exported biquad cascade
that carries `[x1,x2,y1,y2]` state across `processBlock` calls the same way
the LSTM plugin carries hidden state — cheap enough per block to call
synchronously on the audio thread, no worker thread needed unlike the
TCN-reverb pattern below).

**From an arXiv paper + companion GitHub repo.** When the user points at a
paper (e.g. `arxiv:2309.02265`, PESTO) and its code (e.g.
`https://github.com/SonyCSLParis/pesto`):
1. `graphify add https://arxiv.org/abs/<id>` — ingests the paper's
   abstract/metadata into the graph. **Use the `https://arxiv.org/abs/<id>`
   form, not the bare `arxiv:<id>` scheme** — confirmed the CLI's URL
   validator rejects `arxiv:` outright ("Blocked URL scheme 'arxiv' — only
   http and https are allowed"), even though the scheme is used informally
   elsewhere in this doc and in conversation to refer to a paper.
2. `graphify clone <github-url>` — clones the companion repo locally, then run
   AST + semantic extraction on it and merge it into the graph as its own
   repo namespace (see `## graphify` below — this graph is a cross-repo merged
   graph; use the manual JSON-union approach documented there, not
   `graphify merge-graphs`, which corrupts existing repo tags on this graph).
3. Read the cloned repo's actual source directly (its README, its inference/
   conversion scripts, any exported model file) to learn the real calling
   convention — don't assume it matches the paper's abstract.
4. Scaffold a new plugin per the prompt-based path above, and build the
   processor around whatever the model actually needs (sample rate, block
   size, stateful vs stateless, named methods vs bare `forward()`, etc.).
   Real-time-safety pattern: decouple audio-thread `processBlock` from model
   inference via a lock-protected queue + a background `juce::Thread` worker;
   never call `torch::jit::script::Module` methods directly on the audio
   thread if inference could be slow.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cerkut/SMC-JUCE-MCP](https://github.com/cerkut/SMC-JUCE-MCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
