---
trigger: always_on
description: This is **not** a simulation of the human world. It is a computational ecosystem where autonomous agents evolve their own ways of being. No human-world assumptions are injected: no language, no economy, no society templates. Agents live on an abstract graph, perceive through evolved sensory channels, act through continuous effectors, and communicate through brain-driven broadcasts. Everything — brains, senses, drives, signals, memory, motor patterns, cortex reliance — is evolvable. Natural selec
---

# Werld — Project Reference

## Design Philosophy

This is **not** a simulation of the human world. It is a computational ecosystem where autonomous agents evolve their own ways of being. No human-world assumptions are injected: no language, no economy, no society templates. Agents live on an abstract graph, perceive through evolved sensory channels, act through continuous effectors, and communicate through brain-driven broadcasts. Everything — brains, senses, drives, signals, memory, motor patterns, cortex reliance — is evolvable. Natural selection is the only teacher.

The goal is **open-ended evolution**: agents should be able to naturally advance beyond any initial constraints. No part of their cognition is hardcoded. The simulation is designed to run indefinitely on persistent storage.

---

## Project Overview

A Python simulation of autonomous agents that live, perceive, act, reproduce, and evolve on a graph-based substrate. Agents possess **NEAT-style evolvable neural networks**, an optionally-active associative cortex, episodic memory with evolvable parameters, heritable genomes with evolvable drives and I/O dimensions, evolvable sensory processing (64 channels including 19 latent slots), continuous motor effectors with evolvable broadcast bandwidth (up to 16 channels), and the ability to discover compound motor patterns with genome-gated thresholds.

The **Werld Observatory** is a Next.js dashboard providing real-time god-like analysis with 13 sections (Welcome, Methods, Overview, Story, World Map, Population, Evolution, Brain, Intelligence, Ecology, Resources, Communication, Agents), a story generator, world map visualization, pervasive tooltips, and a simulation uptime timer.

The simulation is written in **pure Python** (no ML frameworks). The dashboard is **Next.js / React / TypeScript** with **Recharts** and **shadcn/ui**, reading from the same SQLite database.

---

## Naming

- **Project name**: Werld
- **Platform name**: Werld Observatory

---

## Architecture

```
world_v2/
├── main.py                  # Entry point — CLI, config, SIGTERM handler, watchdog
├── config.py                # All tunable simulation parameters (centralized)
├── claude.md                # This file — comprehensive project reference
├── .gitignore               # Excludes __pycache__, data/, .env, dashboard build artifacts
├── engine/
│   ├── simulation.py        # Core loop, checkpointing, pruning, safeguards, story gen
│   └── substrate.py         # Graph world (Watts-Strogatz), pheromones, seasons, vis coords
├── agents/
│   ├── agent.py             # Central agent class (perceive → decide → learn), 64-input/23-output
│   ├── state.py             # Mutable agent state (energy, entropy, node_id, age)
│   ├── genome.py            # Heritable traits (29) + NEAT node/connection genes + sensory genes
│   ├── cortex.py            # Associative weight table with evolvable drive biases + resolution
│   └── memory.py            # Episodic memory (bounded, importance-pruned, evolvable decay)
├── reasoning/
│   ├── neat_brain.py        # NEAT neural network — evolvable topology, metabolic cost
│   └── brain.py             # Legacy NativeBrain (deprecated, file kept for compat)
├── systems/
│   ├── actions.py           # Continuous effector interpretation + legacy action dispatch
│   ├── signals.py           # Signal propagation (BFS on graph, variable-width vectors)
│   ├── forking.py           # Reproduction, crossover, knowledge transfer, genome-gated macros
│   ├── evolution.py         # Motor pattern (macro) discovery with evolvable capacity/length
│   └── entropy.py           # Entropy accumulation system
├── persistence/
│   ├── db.py                # SQLite schema, connection, compaction, story_chapters table
│   ├── event_log.py         # Logging births, deaths, stats, snapshots
│   ├── state_store.py       # Gzipped JSON checkpoint save/load/milestones
│   └── story.py             # Story chapter generation + substrate topology persistence
├── utils/
│   ├── logger.py            # Console output formatting
│   └── events.py            # Event helper utilities
├── data/
│   ├── simulation.db        # SQLite database (live simulation data)
│   ├── checkpoints/         # Rotating gzipped JSON checkpoints
│   └── milestones/          # Permanent milestone checkpoints (never deleted)
└── dashboard/               # Next.js observatory dashboard
    ├── src/
    │   ├── app/
    │   │   ├── page.tsx             # Main page (sidebar + 13-section routing + uptime timer)
    │   │   ├── layout.tsx           # Root layout (fonts, light theme, "Werld Observatory" title)
    │   │   ├── globals.css          # Light theme CSS variables
    │   │   └── api/simulation/
    │   │       └── route.ts         # API endpoint — all simulation data + start time
    │   ├── components/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nocodemf/werld](https://github.com/nocodemf/werld) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
