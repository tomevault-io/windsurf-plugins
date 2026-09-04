---
trigger: always_on
description: **This file is for AI coding agents** (Claude Code, Codex, Cursor, custom agent harnesses) working inside a fresh clone of OmniSim. It tells you exactly what to do to get a demo running. The conventions below match the [`AGENTS.md` open standard](https://agents.md/), and tools that respect that standard will load this file automatically.
---

# AGENTS.md — Running OmniSim as an AI Coding Agent

**This file is for AI coding agents** (Claude Code, Codex, Cursor, custom agent harnesses) working inside a fresh clone of OmniSim. It tells you exactly what to do to get a demo running. The conventions below match the [`AGENTS.md` open standard](https://agents.md/), and tools that respect that standard will load this file automatically.

If you are a human, this file also works as a quick "what does OmniSim do and how do I run a demo" cheat sheet — the [README](README.md) and the [Developer Quickstart](docs/developer/quickstart.md) have the long form.

---

## 0. For agents: read this first (60 seconds)

**Call it OmniSim.** The simulator is OmniSim — its own product, with substantial additions over the upstream Webots engine it forked from (URDF importer, agent-facing HTTP harness, capture / cinema pipeline, OmniLink agent runtime, omniworld procedural world generator, RL pipeline, CUDA granular-physics, multi-instance parallel runs, OmniSim Wire Protocol). When you talk to the user about the running simulator, the binary, the env vars, the URL scheme — say *OmniSim*. Use "Webots" only when explicitly referring to upstream (the GitHub repo, the file-format syntax inherited from VRML, the original PROTO conventions).

A scenario = one `.wbt` world + one or more controllers + (optionally) a long-running harness or bridge service that you, the agent, drive over HTTP. **You** drive the simulator — you do not need to ask the user to launch, reload, or step it.

### Run this first turn

```bash
python -m omnisim doctor
```

Reports the truth about *this clone right now*: OmniSim binary path, **whether there is a physics runtime at all**, engine↔libController ABI compatibility (the IPC-nonce gate, commit `6eea9d76` — a controller lib older than the engine silently hangs *every* controller at zero ticks while a headless run still prints PASS), port status (`6789` harness, `6790` supervisor, `6791` capture), worlds present, recent commits. Don't guess at the state — check it. `--json` for machine-readable.

**Read the VERDICT line and branch on it before anything else in this file.** Two answers mean STOP:

- `binary NOT FOUND` — this clone is not built. `msys64/` is gitignored, so a fresh `git clone` has no engine and every row of the table below assumes one. Build first (§2): `build_omni.bat` on Windows, `bash scripts/install/linux_bootstrap.sh` on Linux, then `make -C src/omnisim bundle-newton-runtime`.
- `physics FAIL` — the Newton runtime is absent. Newton is the ONLY backend, so this is not a degraded mode: nothing falls, nothing collides, no grasp holds, and every run still exits 0. `doctor` prints the fix for your platform.

`doctor` exits non-zero on either, so an agent branching on `$?` gets the right answer. It did not until 2026-08-28 — it reported the fault and exited 0.

### First moves by task type

Each row states the FINAL rule; the full row (measurements, dates, history) is verbatim in [docs/developer/agents-first-moves.md](docs/developer/agents-first-moves.md), one section per row.

| User asks for | Your first move |
|---|---|
| **You are in an MCP client** (Claude Code, Cursor, Claude Desktop) and can see `omnisim` tools | `harness_status` first; unreachable → `python -m omnisim harness` (it reports on the HARNESS, not the engine). Then `load_world {"path": ..., "light": true}` → `get_scene_tree` → `frame` → `screenshot`. [full](docs/developer/agents-first-moves.md#mcp-client) |
| **Run / see a demo** | `python -m omnisim demo` / `demos`; one world: `python -m omnisim run-world <w>.omniworld` or `run-headless <w>.omniworld` (omit `--duration` unless the run must WATCH). §3. [full](docs/developer/agents-first-moves.md#run-a-demo) |
| **Make a legged robot do a motion** (walk, gait styling, expressive motion — any legged robot) | ⭐ **SHADOWING**: [`projects/policies/training/README.md`](projects/policies/training/README.md), [ghost-design-rules.md](docs/developer/ghost-design-rules.md), [rl-current-state.md](docs/developer/rl-current-state.md). ⚠️ The G1 walk is on a WEIGHT-BEARING balance harness — never "free-standing". [full](docs/developer/agents-first-moves.md#legged-motion) |
| **Make a *reusable skill*, or compose skills into a demo** (walk, turn, carry, stand, climb → a BATON sequence) | ⭐ **SKILL LIBRARY**: `python -m omnisim policy list` / `sequence <name>` / `preview` / `train` / `verify-demos` (`policy --help` is the live list). [skill-library.md](docs/developer/skill-library.md). [full](docs/developer/agents-first-moves.md#skill-library) |
| **Author or edit a world (`.omniworld`)** | ⚠️ `.omniworld`, never `.wbt`. `python -m omnisim harness` → **`POST /world/load {"path": ..., "light": true}`** → edit → **`POST /world/sync`** (325 ms pose-only, 2818 ms structural) → `POST /world/screenshot`. §5. [full](docs/developer/agents-first-moves.md#author-a-world) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [omnilink-tech/omnisim](https://github.com/omnilink-tech/omnisim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-04 -->
