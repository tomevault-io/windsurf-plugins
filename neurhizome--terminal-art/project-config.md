---
trigger: always_on
description: This file documents the codebase structure, development conventions, and workflows for AI assistants working in this repository.
---

# CLAUDE.md — terminal-art (asciicology)

This file documents the codebase structure, development conventions, and workflows for AI assistants working in this repository.

---

## Spirit of the Work

This project is a record of emergence documented from within the process itself. Before reading API signatures or directory trees, understand the methodology:

**Run it. Watch what happens. Write down what surprised you.**

The experiments don't have predetermined outcomes. They are genuine encounters with complexity — patterns that form without being designed, boundaries that emerge from math no walker planned. The sessions in `docs/_posts/` are not tutorials. They are evidence that certain behaviors are possible. Running the same code with different initial conditions will produce different results, and that is correct.

Several constants throughout the codebase are intentionally imprecise — diffusion coefficients that average to almost-right but not quite right. If you correct them, the system dies: equilibrium, gray, nothing to see. The living behavior exists in the gap between correct and almost-correct. **Do not fix these unless you have confirmed they are actual bugs.** Many are the design principle.

The order in which components are composed matters. The initialization history is present in every subsequent state. You cannot cleanly separate the path from the destination.

---

## Self-Documenting Knowledge Graph

The most important automation in this repo: the knowledge graph rebuilds itself.

`tools/graph_viz.py` parses YAML frontmatter from every post and concept file, builds a graph of nodes and edges from the `related:` links, and renders a 96×N ANSI canvas to `docs/assets/captures/knowledge-graph.ans`. This file is displayed live on the `/graph/` page of the blog.

**The graph runs automatically:**

- **Local commits**: `.git/hooks/post-commit` detects changes to `docs/_posts/` or `docs/concepts/` and runs `graph_viz.py`, then stages the updated `.ans` for the next commit.
- **Merge to main**: `.github/workflows/pages.yml` runs `graph_viz.py` before Jekyll builds the site, so the deployed graph always reflects the current state.

The graph is not a map of finished ideas. It is a record of introductions — what met what, in which order. The nodes matter less than the edges. The edges matter less than the sequence.

**To run manually:**
```bash
python3 tools/graph_viz.py
```

**To install the local hook** (if working in a fresh clone):
```bash
cp scripts/hooks/post-commit .git/hooks/post-commit
chmod +x .git/hooks/post-commit
```

**Layout is fully dynamic.** The graph positions every node automatically based on type and date — no hardcoded coordinates. New posts appear in the left (chronological) column; new concepts or aesthetic posts appear in the right (concept space) column. Add a post, commit it, and the graph updates.

---

## Repository Structure

```
terminal-art/
├── src/                          # Core modular toolkit (~3,073 LOC)
│   ├── automata/                 # Walker entities and population management
│   │   ├── walker.py             # Base Walker class (position + genome + age)
│   │   ├── spawner.py            # Population manager (add/remove/breed walkers)
│   │   └── behaviors.py         # Pluggable movement strategies
│   ├── genetics/                 # Memetic trait system
│   │   ├── genome.py             # HSV color genome with inheritance
│   │   ├── inheritance.py        # Parent→child trait flow
│   │   └── speciation.py         # Reproductive barriers (hue distance)
│   ├── fields/                   # 2D grid-based spatial systems
│   │   ├── base.py               # Abstract Field interface
│   │   ├── diffusion.py          # Scent trails / chemical diffusion
│   │   ├── territory.py          # Chunked ownership tracking
│   │   └── energy.py             # Excitable medium with cascade dynamics
│   ├── events/                   # Temporal perturbation system
│   │   ├── event.py              # Base Event class
│   │   ├── catalog.py            # Pre-built event library (AESTHETIC_POOL, CHAOS_POOL)
│   │   └── scheduler.py          # Event timing and triggering
│   ├── glyphs/                   # Probabilistic Unicode character selection
│   │   ├── picker.py             # GlyphPicker (load DB, query by direction/style)
│   │   └── direction.py          # Direction enum (N, NE, E, SE, S, SW, W, NW, NONE)
│   ├── renderers/                # Terminal display layer
│   │   └── terminal_stage.py     # TerminalStage: double-buffered full-screen canvas
│   └── utils/
│       └── colors.py             # HSV/RGB conversion, circular hue mean
├── experiments/                  # Composable experiment scripts (reference implementations)
│   ├── simple_walkers.py         # Minimal example: walkers + colors (~30 lines)
│   ├── memetic_territories.py    # Full-featured: genetics + fields + events
│   ├── color_speciation.py       # Reproductive barriers creating color species
│   ├── gradient_flow.py          # Aesthetic mode: flowing color gradients
│   ├── predator_prey.py          # Lotka-Volterra dynamics (green prey vs red predators)
│   └── README.md                 # Experiment composition guide

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [neurhizome/terminal-art](https://github.com/neurhizome/terminal-art) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
