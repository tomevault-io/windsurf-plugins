---
trigger: always_on
description: Notes for agents working on this repo.
---

# AGENTS.md, pokelike.xyz.bot

Notes for agents working on this repo.

**Read [README.md](README.md) first.** It is the human tour: what the project
does, how it is installed and used. This file adds what someone *changing* the
code needs, internals, pitfalls, and the reasoning behind decisions that look
odd, and the per-folder `AGENTS.md` files add the specifics of each area:

- **[bots/AGENTS.md](bots/AGENTS.md)**, the bot competition: how a folder becomes
  a bot, the fingerprint, self-containment, the LLM harness knobs and seams.
- **[experiments/AGENTS.md](experiments/AGENTS.md)**, the research area: the MDP,
  the reward registry, what is tracked and why, measuring a candidate by path.
- **[llm-bench/AGENTS.md](llm-bench/AGENTS.md)**, the model benchmark: the frozen
  harnesses, the seven-key fingerprint, what a pass writes.

The human-facing counterpart of each is the `README.md` beside it.

**Orientation**
- [What this is](#what-this-is)
- [Commands](#commands)
- [Architecture](#architecture)
- [Design overview](#design-overview)

**How it works**
- [Talking to the game](#talking-to-the-game)
- [The HTTP API](#the-http-api)
- [Scoring](#scoring)
- [Reproducibility](#reproducibility)
- [Performance](#performance)

**Before you change anything**
- [Real pitfalls](#real-pitfalls)
- [Tests](#tests)
- [The frozen harnesses](#the-frozen-harnesses)
- [Secrets](#secrets)

---

## What this is

An environment for letting bots play [pokelike.xyz](https://pokelike.xyz/), a
Pokémon roguelike that runs entirely in the browser. The game has no backend: all
its logic is in one obfuscated JavaScript bundle. We run it in headless Chromium
and talk to its global functions.

## Commands

```bash
uv sync                          # environment
uv run pokelike setup            # browser + offline copy (once)
uv run pokelike play --seed 42   # interactive run
uv run pokelike play --region johto      # any of kanto, johto, hoenn, sinnoh
uv run pokelike bot run --bot mine --regions all   # all four, stopping at a loss
uv run pokelike schema           # what a bot receives, printed from a live game
uv run pokelike history -d       # what you played here, columns explained
uv run pytest                    # full suite, ~1 minute, needs the game on disk
uv run pytest -m "not slow"      # 163 tests in 6 s, no browser. What CI runs

# THE COMPETITION: your code is the entry, the game is fixed
uv run pokelike bot new mine                        # a folder that already plays
uv run pokelike bot new mine --llm                  # ... from the shared LLM harness
uv run pokelike bot run --bot random --runs 5       # play it
uv run pokelike bot run -d --runs 1                 # log every decision
uv run pokelike bot bench --bot random              # the 50 standard seeds, records
uv run pokelike bot bench --bot random --dry-run    # ... without writing an entry
uv run pokelike bot board                           # the standings, from disk

# THE MODEL BENCHMARK: the scaffold is frozen, the model is the entry
uv run pokelike model bench --harness v4 --model a/b
uv run pokelike model bench --harness v4 --model a/b --set notes=4  # smaller notebook
# --set goes to the harness constructor, which refuses by name what it does not
# know. Shared flags are the same for every version; a version's own knob is its own.
uv run pokelike model board --harness v4            # that version's table
uv run pokelike model watch                         # follow the running pass
uv run pokelike model watch --stamp 20260820-1533   # pick one, when several run
uv run pokelike model watch --all                   # every pass on this machine
# --harness is REQUIRED on both: a version is the question a row answers
# credentials: .env at the repo root, then $FW_ENDPOINT/$FW_TOKEN/$MODEL_ID, then
# --endpoint/--api-key/--model. Later always wins, and .env is the one to use:
# a key on a command line is in ps for every user and in your shell history
#
# The flat names `bench`, `leaderboard`, `llm-bench` and `new-bot` were removed,
# not aliased, and there is no implicit verb: `pokelike bot` alone is an error.

uv run python -m experiments.example.train --episodes 20     # the shape of one
uv run python -m experiments.sarsa.train --episodes 300      # the real thing
uv run pokelike bot bench --bot experiments/mine --dry-run   # measure a candidate
```

Two ports, kept distinct: the **asset server** (the game files, served to the
headless browser) defaults to **8422** (`--port`); the **HTTP JSON API**
(`pokelike api`) defaults to **8423** (`--api-port`).

## Setup on minimal Linux

`pokelike setup` downloads the browser and the offline game and checks the browser
actually launches. On minimal images (Raspberry Pi, servers, containers) Chromium's
system libraries are often missing, and `playwright install` exits 0 even when they are,
it only warns, so `setup` launches the browser to check rather than trusting the exit
code. When that is the case it prints the fix:

```bash
sudo $(which python) -m playwright install-deps chromium
```

Use `sudo $(which python)`, not plain `sudo playwright`: the virtualenv is not on root's
PATH. There is no environment to activate, `uv run` handles it; `source
.venv/bin/activate` and then dropping the `uv run` prefix works too.

## Architecture


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pierpierpy/pokelike.xyz.bot](https://github.com/pierpierpy/pokelike.xyz.bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
