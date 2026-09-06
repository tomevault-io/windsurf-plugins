---
trigger: always_on
description: **ShopGym** is a mono-repo for building and evaluating shopping LLM agents. It contains four packages:
---

# AGENTS.md

**ShopGym** is a mono-repo for building and evaluating shopping LLM agents. It contains four packages:

- `packages/shop_arena` (Python) — **ShopArena**: Environment Factory that generates deterministic, self-contained sandbox shops (**SandboxShops**) from live storefronts. Ships three top-level modules: `shop_arena.gen` (generation pipeline), `shop_arena.explore` (storefront exploration), and `shop_arena.env_eval` (per-shop environment-quality measurement; CLI: `shop-env-eval`).
- `packages/shop_guru` (Python) — **ShopGuru**: automated dataset generation pipeline that ingests a sandbox shop's catalog, navigation, and policies to synthesize grounded evaluation tasks across 7 skill categories.
- `packages/shop_backend` (TypeScript) — **ShopBackend**: local GraphQL API server hosting SandboxShop data; used for benchmarking and as an RL environment.
- `packages/harness` (Python) — **Harness**: runtime-agnostic plan-then-loop engine that orchestrates LLM agents. Owns process lifecycle, workspace state, and telemetry; prompts and tools come from the chosen `AgentRuntime` adapter (e.g. Claude Code, `pi`).

IMPORTANT: Always refer to `README.md` for project context, setup and common commands. Always keep `README.md` up-to-date.

Python packages are managed with `uv` (workspace at repo root). TypeScript packages are managed with `pnpm` workspaces.

## Repository Structure

```
shop-gym/
├── AGENTS.md                   # this file — agent behavior + coding guidelines
├── CLAUDE.md -> AGENTS.md      # alias for Claude Code
├── README.md                   # project overview, setup, quickstart
├── packages/
│   ├── shop_arena/             # Python — SandboxShop factory (gen / explore / env_eval)
│   ├── shop_guru/              # Python — dataset generation + eval pipeline
│   ├── shop_backend/           # TypeScript — GraphQL API server for SandboxShops
│   └── harness/                # Python — runtime-agnostic plan-then-loop agent engine
├── docs/
│   └── specs/                  # design specifications (intent — see `docs/specs/README.md`)
├── scripts/                    # repo-wide helpers (e.g. `run-shop.sh`)
├── notebooks/                  # exploratory analysis
├── outputs/                    # generated artifacts: shop_manuals, shops, shop_guru, shop_env_evals
├── pyproject.toml              # uv workspace root
├── pnpm-workspace.yaml         # pnpm workspace root
├── tsconfig.base.json          # shared TS compiler config
└── biome.json                  # TS lint/format config
```

Typical data flow: `shop_arena.explore` → `shop_arena.gen` → SandboxShop served by `shop_backend` → `shop_guru` synthesizes tasks and evaluates agents. The `harness` package provides the runtime-agnostic engine used by the build and eval loops.

## Behavior Guideline

### Use Specifications

IMPORTANT: Before implementing any feature, consult the specifications in `docs/specs/README.md`.

- Assume NOT implemented. Many specs describe planned features that may not yet exist in the codebase.
- Check the codebase first. Before concluding something is or isn't implemented, search the actual code. Specs describe intent; code describes reality.
- Use specs as guidance. When implementing a feature, follow the design patterns, types, and architecture defined in the relevant spec.
- Spec index: `docs/specs/README.md` lists all specifications organized by category (core, LLM, security, etc.).
- For designing new features, create the pure specifications in `docs/specs/<feature>.md`, and track implementation plans in issues or pull requests.
- During implementation, make docstrings self-contained and DO NOT refer to the specs.

Each specification / design doc need to follow the structure:

```
 Overview
 Terminology
 Current Status
 Desired Status
 Proposal
 Alternative (optional, if any)
 Execution Table
 Appendix
```

### Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:

- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

### Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

### Hierarchy Matters

When create new files:

- Think about the big picture and maintain a good codebase structure.
- Don't put every files under the same folder. Instead, build modules with high cohesion, low coupling.

### Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:

- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agentic-foundation-modeling-research/shop-gym](https://github.com/agentic-foundation-modeling-research/shop-gym) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
