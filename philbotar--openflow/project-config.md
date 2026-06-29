---
trigger: always_on
description: Single-file orientation for contributors and coding agents.
---


# AGENTS.md

Single-file orientation for contributors and coding agents.

## Crate orientation (AGENTS.md)

| Crate | File |
| --- | --- |
| engine | [`crates/engine/AGENTS.md`](crates/engine/AGENTS.md) |
| providers | [`crates/providers/AGENTS.md`](crates/providers/AGENTS.md) |
| orchestration | [`crates/orchestration/AGENTS.md`](crates/orchestration/AGENTS.md) |
| desktop | [`crates/desktop/AGENTS.md`](crates/desktop/AGENTS.md) |
| ui | [`crates/ui/AGENTS.md`](crates/ui/AGENTS.md) |

Each file covers architecture, dependency rules, code standards, patterns, and change checklists for that crate.

## 30-Second Intake

1. This is a Rust workspace with five crates: `engine`, `providers`, `orchestration`, `desktop`, `ui`.
2. Core rule: keep engine logic in `engine`; keep API transport/auth quirks in `providers`; keep runtime/state/storage in `orchestration`; keep Tauri adapter code in `desktop`; keep frontend code in `ui`.
   - **engine** — valid workflow + run behavior
   - **orchestration** — store, load, wire, host runs
   - **providers** — LLM transport
   - **ui** / **desktop** — user interaction
3. Start docs at `docs/README.md` — see [Documentation](#documentation) for the full tree.
4. Development lanes: `docs/contributing/development-lanes.md`.
5. Coding patterns: `docs/contributing/coding-patterns.md`.
6. Workflow verification: `docs/contributing/testing-workflows.md`.
7. Engine vocabulary: `docs/glossary.md`.

## Boundary Seams

Add a port/trait only when a consumer is typed on that interface. Otherwise call the concrete type directly.

| Seam | Location |
| --- | --- |
| LLM invocation (`AiPort`, `AgentRequest`) | `crates/engine/src/ports/outbound.rs` |
| Tool and subagent execution (`ToolPort`) | `crates/engine/src/ports/outbound.rs` → `crates/orchestration/src/run/execution/tool_port.rs` |
| Human input / tool approval | `crates/engine/src/ports/inbound.rs` |
| Provider client (`AiClient: AiPort`) | `crates/providers/src/client.rs` |
| UI → desktop IPC | `crates/ui/src/port.ts` (`UiDesktopOutboundPort`) |

## Documentation

```text
docs/
├── README.md
├── glossary.md
├── getting-started/
│   └── README.md
├── guides/
│   └── first-workflow.md
├── concepts/
│   ├── README.md
│   ├── how-openflow-works.md
│   └── workflows-and-runs.md
├── reference/
│   └── README.md
├── troubleshooting/
│   └── README.md
├── contributing/
│   ├── README.md
│   ├── development-lanes.md
│   ├── coding-patterns.md
│   └── testing-workflows.md
└── architecture/
    ├── README.md
    ├── contract.md
    ├── threading-concurrency.md
    └── diagrams/
```

| Doc | Use when |
| --- | --- |
| `docs/README.md` | First read; filesystem index |
| `docs/getting-started/README.md` | Running the app and configuring a provider |
| `docs/guides/first-workflow.md` | Building the first workflow |
| `docs/concepts/how-openflow-works.md` | Understanding the runtime path |
| `docs/reference/README.md` | Commands, storage paths, provider key resolution |
| `docs/troubleshooting/README.md` | Setup, provider, run, and verification failures |
| `docs/contributing/development-lanes.md` | Classifying a change, selecting playbook/skill, choosing verification |
| `docs/contributing/coding-patterns.md` | Ownership, runtime semantics, conventions |
| `docs/contributing/testing-workflows.md` | Acceptance tests, live-AI smoke |
| `docs/architecture/contract.md` | Layer boundaries and dependency rules |
| `docs/architecture/threading-concurrency.md` | Runtimes, async I/O, parallelism |
| `docs/glossary.md` | Engine terms and naming |

## Development Lanes

Before editing, classify the change with [`docs/contributing/development-lanes.md`](docs/contributing/development-lanes.md). Agent skills and editor rules should route to that doc instead of carrying their own copy of architecture facts.

| Touched area | Lane | Local guide |
| --- | --- | --- |
| `crates/engine/**` | Engine semantics | `crates/engine/AGENTS.md` |
| `crates/orchestration/src/run/**` | Run orchestration | `crates/orchestration/AGENTS.md` |
| `crates/orchestration/src/{agent,workflow,project,settings,tool}/**` | Application/domain service | `crates/orchestration/AGENTS.md` |
| `crates/orchestration/src/adapters/**` | Concrete adapter/I/O | `crates/orchestration/AGENTS.md` |
| `crates/providers/**` | Provider adapter | `crates/providers/AGENTS.md` |
| `crates/desktop/**` | Desktop IPC adapter | `crates/desktop/AGENTS.md` |
| `crates/ui/**` | UI/Desktop seam and presentation | `crates/ui/AGENTS.md` |

## Repo Map

### Workspace and engine

| Path | Purpose | Change Here When... |
| --- | --- | --- |
| `Cargo.toml` | Workspace members and shared dependencies | Adding crates or shared dep versions |
| `crates/engine/src/graph/` | Workflow model, `WorkflowSettings`, node config, `CallableAgent`, DAG validation | Changing schema, graph rules, or scheduling |
| `crates/engine/src/execution/workflow_runner.rs` | Non-interactive `WorkflowRunner` | Changing batch run semantics |
| `crates/engine/src/execution/interactive_engine.rs` | Interactive engine `poll()` + `run()` loop | Changing pause/resume or self-driving run behavior |
| `crates/engine/src/execution/subagent_runtime.rs` | Subagent declare/call builtins + turn machine | Changing subagent invocation semantics |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [philbotar/OpenFlow](https://github.com/philbotar/OpenFlow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
