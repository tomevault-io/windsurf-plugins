---
trigger: always_on
description: Automaton behavioral contracts for vibe-graph. These define per-module stability, roles, and change impact rules.
---


# Behavioral Contracts: vibe-graph

Each module in this codebase has a role, stability level, and behavioral rules.
AI agents and developers should respect these contracts when making changes.

## Defaults

- Default rule: `identity`
- Damping coefficient: 0.5
- Inheritance mode: Compose

## Role: `identity`

Nodes: 101

| Path | Stability | Impact |
|------|-----------|--------|
| `release.toml` | 0.32 | — |
| `Cargo.toml` | 0.32 | — |
| `crates/vibe-graph-core/Cargo.toml` | 0.32 | — |
| `crates/vibe-graph-layout-gpu/Cargo.toml` | 0.32 | — |
| `crates/vibe-graph-layout-gpu/README.md` | 0.32 | — |
| `crates/vibe-graph-layout-gpu/examples/large_graph.rs` | 0.32 | — |
| `crates/vibe-graph-layout-gpu/examples/simple_layout.rs` | 0.32 | — |
| `crates/vibe-graph-layout-gpu/src/quadtree.rs` | 0.32 | — |
| `crates/vibe-graph-layout-gpu/src/layout.rs` | 0.32 | — |
| `crates/vibe-graph-layout-gpu/src/error.rs` | 0.44 | — |
| `crates/vibe-graph-layout-gpu/src/shaders.rs` | 0.32 | — |
| `crates/vibe-graph-layout-gpu/src/gpu.rs` | 0.33 | — |
| `crates/vibe-graph-viz/Cargo.toml` | 0.32 | — |
| `crates/vibe-graph-viz/index.html` | 0.32 | — |
| `crates/vibe-graph-viz/Makefile` | 0.32 | — |
| `crates/vibe-graph-viz/README.md` | 0.32 | — |
| `crates/vibe-graph-viz/examples/native.rs` | 0.32 | — |
| `crates/vibe-graph-viz/examples/automaton_viz.rs` | 0.32 | — |
| `crates/vibe-graph-viz/examples/run_automaton_viz.sh` | 0.32 | — |
| `crates/vibe-graph-viz/Trunk.toml` | 0.32 | — |
| `crates/vibe-graph-viz/src/ui/overlays.rs` | 0.32 | — |
| `crates/vibe-graph-viz/src/automaton_mode.rs` | 0.32 | — |
| `crates/vibe-graph-viz/src/render.rs` | 0.32 | — |
| `crates/vibe-graph-viz/src/gpu_layout.rs` | 0.32 | — |
| `crates/vibe-graph-viz/src/automaton_app.rs` | 0.32 | — |
| `crates/vibe-graph-viz/src/selection.rs` | 0.32 | — |
| `crates/vibe-graph-viz/src/git_panel.rs` | 0.32 | — |
| `crates/vibe-graph-viz/src/settings.rs` | 0.33 | — |
| `crates/vibe-graph-viz/src/sample.rs` | 0.32 | — |
| `crates/vibe-graph-viz/src/api.rs` | 0.35 | — |
| `crates/vibe-graph-viz/src/top_bar.rs` | 0.32 | — |
| `crates/vibe-graph-ui/Cargo.toml` | 0.32 | — |
| `crates/vibe-graph-ops/Cargo.toml` | 0.32 | — |
| `crates/vibe-graph-ops/src/error.rs` | 0.32 | — |
| `crates/vibe-graph-ops/src/config.rs` | 0.32 | — |
| `crates/vibe-graph-ops/src/store.rs` | 0.32 | — |
| `crates/vibe-graph-ops/src/responses.rs` | 0.33 | — |
| `crates/vibe-graph-ops/src/project.rs` | 0.35 | — |
| `crates/vibe-graph-ops/src/workspace.rs` | 0.33 | — |
| `crates/vibe-graph-ops/src/requests.rs` | 0.33 | — |
| `crates/vibe-graph-ops/src/scan.rs` | 0.32 | — |
| `crates/vibe-graph-ops/src/context.rs` | 0.32 | — |
| `crates/vibe-graph-api/Cargo.toml` | 0.32 | — |
| `crates/vibe-graph-api/tests/api_integration.rs` | 0.32 | — |
| `crates/vibe-graph-api/src/types.rs` | 0.32 | — |
| `crates/vibe-graph-api/src/routes/health.rs` | 0.32 | — |
| `crates/vibe-graph-api/src/routes/graph.rs` | 0.32 | — |
| `crates/vibe-graph-api/src/routes/git.rs` | 0.32 | — |
| `crates/vibe-graph-api/src/routes/ops.rs` | 0.32 | — |
| `crates/vibe-graph-api/src/ws.rs` | 0.32 | — |
| `crates/vibe-graph-engine/Cargo.toml` | 0.32 | — |
| `crates/vibe-graph-constitution/Cargo.toml` | 0.32 | — |
| `crates/vibe-graph-git/Cargo.toml` | 0.32 | — |
| `crates/vibe-graph-semantic/Cargo.toml` | 0.32 | — |
| `crates/vibe-graph-ssot/Cargo.toml` | 0.32 | — |
| `crates/vibe-graph-materializer/Cargo.toml` | 0.32 | — |
| `crates/vibe-graph-llmca/Cargo.toml` | 0.32 | — |
| `crates/vibe-graph-llmca/src/prompt_rule.rs` | 0.32 | — |
| `crates/vibe-graph-cli/Cargo.toml` | 0.32 | — |
| `crates/vibe-graph-cli/LICENSE` | 0.32 | — |
| `crates/vibe-graph-cli/tests/cli_integration.rs` | 0.32 | — |
| `crates/vibe-graph-cli/README.md` | 0.32 | — |
| `crates/vibe-graph-cli/templates/composer.md` | 0.32 | — |
| `crates/vibe-graph-cli/assets/vibe_graph_viz.js` | 0.32 | — |
| `crates/vibe-graph-cli/assets/vibe_graph_viz_bg.wasm` | 0.32 | — |
| `crates/vibe-graph-cli/src/config.rs` | 0.32 | — |
| `crates/vibe-graph-cli/src/commands/serve.rs` | 0.32 | — |
| `crates/vibe-graph-cli/src/commands/remote.rs` | 0.32 | — |
| `crates/vibe-graph-cli/src/commands/config.rs` | 0.32 | — |
| `crates/vibe-graph-cli/src/commands/compose.rs` | 0.32 | — |
| `crates/vibe-graph-cli/src/commands/automaton.rs` | 0.32 | — |
| `crates/vibe-graph-cli/src/commands/viz.rs` | 0.32 | — |
| `crates/vibe-graph-sync/Cargo.toml` | 0.32 | — |
| `crates/vibe-graph-mcp/Cargo.toml` | 0.32 | — |
| `crates/vibe-graph-mcp/src/types.rs` | 0.44 | — |
| `crates/vibe-graph-mcp/src/tools.rs` | 0.32 | — |
| `crates/vibe-graph-mcp/src/server.rs` | 0.32 | — |
| `crates/vibe-graph-mcp/src/gateway.rs` | 0.32 | — |
| `crates/vibe-graph-automaton/Cargo.toml` | 0.32 | — |
| `crates/vibe-graph-automaton/PLANNING.md` | 0.32 | — |
| `crates/vibe-graph-automaton/tests/integration_tests.rs` | 0.32 | — |
| `crates/vibe-graph-automaton/README.md` | 0.32 | — |
| `crates/vibe-graph-automaton/examples/run_llm_gol.sh` | 0.32 | — |
| `crates/vibe-graph-automaton/examples/llm_game_of_life.rs` | 0.32 | — |
| `crates/vibe-graph-automaton/examples/game_of_life.rs` | 0.32 | — |
| `crates/vibe-graph-automaton/src/source_code.rs` | 0.32 | — |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pinsky-three/vibe-graph](https://github.com/pinsky-three/vibe-graph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
