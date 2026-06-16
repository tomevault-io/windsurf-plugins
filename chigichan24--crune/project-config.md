---
trigger: always_on
description: crune is a static web dashboard that visualizes Claude Code session logs. It consists of two parts:
---

# CLAUDE.md

## Project Overview

crune is a static web dashboard that visualizes Claude Code session logs. It consists of two parts:

1. **Data pipeline** (`scripts/analyze-sessions.ts` + `scripts/knowledge-graph-builder.ts`): Reads JSONL session files, extracts structured data, builds a semantic knowledge graph, and outputs JSON files to `public/data/sessions/`.
2. **Frontend** (`src/`): React SPA that loads the generated JSON and renders three views --- Overview, Session Playback, and Knowledge Graph.

## Build & Run

```bash
npm install
npm run analyze-sessions   # Generate data from ~/.claude/projects/
npm run dev                # Vite dev server at localhost:5173
npm run build              # tsc -b && vite build -> dist/
npm run lint               # ESLint
npm run skill-server       # Local synthesis server at localhost:3456
npm run dev:full           # skill-server + Vite dev server together
```

## Architecture

- **No routing library** --- `App.tsx` manages view state via `useState<ViewMode>` (overview | playback | knowledge)
- **No state management library** --- Data flows through props from `App.tsx` down. Each view fetches its own data via custom hooks (`useSessionIndex`, `useSessionDetail`, `useSessionOverview`)
- **Plain CSS** --- Each component has a co-located `.css` file. Global CSS variables are in `src/index.css` (colors, fonts, shadows)
- **Session playback** opens as a right-side drawer overlay, not a separate route

## Key Conventions

- UI text is in Japanese (commit `abc92e1`)
- Color variables have semantic roles: `--accent` (amber, interactive), `--brand` (amber, logo only — unified with accent), `--success/warning/danger` (status), `--chart-1..6` (data visualization)
- Tool call display logic is centralized in `ToolCallBlock.tsx` with per-tool rendering (Bash, Edit, Write, Read, Grep, Glob, Agent)
- Expand/collapse pattern: `useState(false)` + conditional render (no animation), see `SubagentBranch.tsx`

## Data Pipeline Details

`analyze-sessions.ts` reads from `~/.claude/projects/` by default. Override with `--sessions-dir` and `--output-dir` flags.

Before analysis, `/insights` is automatically executed via `claude -p` to refresh facets data in `~/.claude/usage-data/facets/`. Use `--skip-facets` to skip this step.

Output structure:
```
public/data/sessions/
  index.json              # All sessions (sorted by createdAt desc)
  overview.json           # Cross-session stats + knowledge graph + tacit knowledge
  detail/{sessionId}.json # Individual session turns, subagents, linked plan
```

The knowledge graph builder (`knowledge-graph-builder.ts`) uses a multi-signal embedding approach:
- Text TF-IDF (weight 0.50) + Tool-IDF (weight 0.25) + structural features (weight 0.25), concatenated with sqrt-weighting
- Truncated SVD via Gram matrix power iteration (k = min(80, max(20, m/4)))
- Agglomerative clustering (average linkage) with elbow-detected threshold + oversized cluster splitting + **facets-based narrow cluster merging**
- Louvain community detection -> Brandes betweenness centrality
- See [docs/knowledge-graph-algorithm.md](docs/knowledge-graph-algorithm.md) for full details

### /insights Integration

The pipeline reads `/insights` facets data (`~/.claude/usage-data/facets/`) to improve accuracy:
- **Topic labels**: Uses `underlying_goal` from facets instead of TF-IDF keywords when available
- **Narrow cluster merging**: Merges small clusters (≤2 sessions) that share normalized goal categories
- **Reusability scoring**: Adds `successRate` and `helpfulness` signals (weight 0.10 each) derived from `outcome` and `claude_helpfulness`
- **Synthesis prompt enrichment**: Includes goals, friction details, and success rate in the LLM synthesis prompt
- **Session list**: Shows `brief_summary` from facets instead of raw first user prompt

Facets reader (`scripts/knowledge-graph/facets-reader.ts`) normalizes 50+ raw goal categories into ~10 canonical categories (feature, bugfix, refactoring, documentation, review, testing, ci, git_ops, setup, other).

## Skill Synthesis

The pipeline detects recurring workflow patterns and generates reusable Claude Code skill definitions using LLM synthesis via `claude -p`.

### Pre-synthesis (build time)

`analyze-sessions` automatically synthesizes the top skill candidates during data generation:

```bash
npm run analyze-sessions                              # Top 5 candidates, default model
npm run analyze-sessions -- --synthesize-model haiku     # Use Haiku for speed
npm run analyze-sessions -- --synthesize-count 3         # Distill only top 3
npm run analyze-sessions -- --skip-synthesize            # Skip synthesis entirely
```

Flags:
- `--synthesize-model <model>` --- Use a specific Claude model (e.g. `haiku`, `sonnet`, `opus`)
- `--synthesize-count <n>` --- Number of top candidates to synthesize (default: 5)
- `--skip-synthesize` --- Skip LLM synthesis for faster builds
- `--facets-dir <path>` --- Custom facets directory (default: `~/.claude/usage-data/facets`)
- `--skip-facets` --- Skip `/insights` refresh and facets integration
- `--use-human-feedback` --- Fold playback feedback (issue #24) into synthesis + reusability scoring (default OFF for A/B)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chigichan24/crune](https://github.com/chigichan24/crune) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
