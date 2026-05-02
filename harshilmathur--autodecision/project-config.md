---
trigger: always_on
description: > Drop this file into your project root alongside the `.claude/` skill directory.
---

# CLAUDE.md — Auto-Decision Engine

> Drop this file into your project root alongside the `.claude/` skill directory.
> Claude Code (and any compatible AI agent) can then use autodecision immediately.

## What is Autodecision?

Iterative decision simulation based on [Karpathy's autoresearch](https://github.com/karpathy/autoresearch) and [LLM Council](https://github.com/karpathy/llm-council). Five independent persona agents debate, critique each other anonymously, and iterate until a Convergence Judge measures mechanical stability. Works on any business or strategic decision.

**Core loop:** Scope → Ground → Elicit → Hypothesize → Simulate (5 personas) → Critique → Adversary → Sensitivity → Converge → Decide.

---

## Installation

### Claude Code

**Plugin (recommended)**

```
/plugin marketplace add harshilmathur/autodecision
/plugin install autodecision@autodecision
```

All commands become available under the `/autodecision:` namespace (the main loop is `/autodecision:autodecision`).

**Legacy skill install**

If you'd rather vendor the skill files directly into a `.claude/` directory:

```bash
git clone https://github.com/harshilmathur/autodecision.git
cd autodecision
./install.sh                      # global: ~/.claude
./install.sh ./your-project/.claude   # project-level
```

Bare `/autodecision` works in this mode (the plugin form requires the `autodecision:` prefix). Restart your Claude Code session to pick up the changes.

### Claude Cowork

**From marketplace (recommended)**

Two steps:
1. **Customize → Create plugin → Add marketplace**, paste `https://github.com/harshilmathur/autodecision`.
2. **Customize → Add plugin → Personal → autodecision → Install**.

Adding the marketplace alone does not install the plugin — step 2 is required.

**From release zip** (offline / restricted network)

Download `autodecision-<version>.zip` from the [latest release](https://github.com/harshilmathur/autodecision/releases/latest), then in Cowork: **Customize → Create plugin → Upload plugin** and select the zip.

---

## Commands

| Command | Purpose | Time |
|---------|---------|------|
| `/autodecision` | Full iterative loop with persona council | ~15-20 min |
| `/autodecision:quick` | Single-pass, no council, no iteration | ~2-3 min |
| `/autodecision:compare` | Side-by-side comparison (fresh or post-facto) | ~5 min |
| `/autodecision:revise` | Revise a previous run with changed assumptions/data/tilt | ~8-10 min |
| `/autodecision:challenge` | Stress-test a proposed action (adversary-only, no full loop) | ~5 min |
| `/autodecision:summarize` | Compress a brief into a shareable one-page summary | ~1 min |
| `/autodecision:publish` | Ship a brief as PDF → Notion, email draft, gist, Slack, Drive, or Local | ~1 min |
| `/autodecision:plan` | Interactive setup wizard (scope only) | ~2 min |
| `/autodecision:review` | Review past decisions, compare predictions vs outcomes | ~1 min |
| `/autodecision:export` | Bundle journal + assumptions into portable archive | ~1 min |

---

## Quick Start

### Analyze a decision (full loop)

```
/autodecision "Should we cut pricing by 20%?"
```

Runs: scope → ground (web search) → elicit (review with user) → 2 iterations of 5-persona council with adversarial pressure → Decision Brief.

### Quick sanity check

```
/autodecision:quick "Should we launch in Southeast Asia?"
```

Single analyst, no council, no iteration. Effects chain output in ~2 minutes.

### Compare two options

```
/autodecision:compare "Cut pricing 20%" vs "Cut pricing 10%"
```

Runs quick mode on both, produces side-by-side comparison table.

### Compare existing runs

```
/autodecision:compare --existing pricing-cut-20pct-full vs market-expansion-full
```

Reads two completed Decision Briefs and compares structurally.

### Use a template

```
/autodecision --template pricing "Should we cut pricing by 20%?"
/autodecision --template expansion "Should we launch in the US?"
/autodecision --template build-vs-buy "Should we build our own auth?"
/autodecision --template hiring "Should we hire a VP Engineering?"
```

Templates pre-populate sub-questions, constraints, and search queries for common decision types.

### Control iteration depth

```
/autodecision --iterations 1 "decision"    # Medium: council, 1 pass, no convergence
/autodecision --iterations 2 "decision"    # Default: full loop
/autodecision --iterations 4 "decision"    # Deep: high-stakes decisions
```

### Review past decisions

```
/autodecision:review                                    # List all past decisions
/autodecision:review pricing-cut-20pct-full             # Show a specific brief
/autodecision:review pricing-cut-20pct-full --outcome "Acquisition increased 25%"
```

### Attach context documents

```
/autodecision "Should we take the Series A?" --context term-sheet.pdf
/autodecision "Should we acquire Acme?" --context financials.csv competitive-analysis.md
```

Attaches files alongside the decision question. The engine extracts key data points, tags them `[D#]`, and threads them through the full pipeline. Supported: `.md`, `.txt`, `.pdf`, `.csv`, `.json`, images. Claude Code only (requires filesystem access).

### Skip the user review step

```
/autodecision --skip-elicit "Should we cut pricing by 20%?"
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [harshilmathur/autodecision](https://github.com/harshilmathur/autodecision) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
