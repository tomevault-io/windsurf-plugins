---
trigger: always_on
description: <!-- nx configuration start-->
---

<!-- nx configuration start-->
<!-- Leave the start & end comments to automatically receive updates. -->

# General Guidelines for working with Nx

- When running tasks (for example build, lint, test, e2e, etc.), always prefer running the task through `nx` (i.e. `nx run`, `nx run-many`, `nx affected`) instead of using the underlying tooling directly
- You have access to the Nx MCP server and its tools, use them to help the user
- When answering questions about the repository, use the `nx_workspace` tool first to gain an understanding of the workspace architecture where applicable.
- When working in individual projects, use the `nx_project_details` mcp tool to analyze and understand the specific project structure and dependencies
- For questions around nx configuration, best practices or if you're unsure, use the `nx_docs` tool to get relevant, up-to-date docs. Always use this instead of assuming things about nx configuration
- If the user needs help with an Nx configuration or project graph error, use the `nx_workspace` tool to get any errors

<!-- nx configuration end-->

# ng-forge Development Guidelines

## Working Style

- **Spawn subagents if you believe the task is better to be divided**
- **Never jump to implementation without explicit user approval.** When asked to analyze, plan, or investigate, stay in analysis mode until the user says to proceed with code changes
- **Always scope changes precisely to what was requested.** Do not apply changes globally when they should be limited to a specific file, component, or example. When in doubt, ask
- **When the user describes a design constraint or architectural principle, treat it as absolute.** Do not attempt to relax, work around, or make stated design principles optional
- **Keep plans concise and correct on the first attempt.** Do not over-engineer solutions. If the user says the plan is too complex, simplify immediately rather than rewriting with similar complexity
- **When adding workarounds like arbitrary delays or skipping tests, flag them as code smells** and propose a proper fix first. Only use workarounds as a last resort with explicit user approval
- **When working with git operations, confirm the target branch, repo directory, and scope** before executing. Never run destructive git operations without confirmation

## Quick Reference

| Command                    | Description                                 |
| -------------------------- | ------------------------------------------- |
| `pnpm install`             | Install dependencies                        |
| `nx test <project>`        | Unit tests (Vitest)                         |
| `nx build <project>`       | Build library                               |
| `nx lint <project>`        | ESLint                                      |
| `nx e2e <project>`         | E2E tests locally (screenshots will differ) |
| `pnpm serve:docs`          | Serve docs app for dev                      |
| `pnpm e2e:material`        | E2E in Docker (screenshots match CI)        |
| `pnpm e2e:material:update` | Update screenshots in Docker                |
| `pnpm e2e:clean`           | Clean Docker E2E cache                      |
| `pnpm build:libs`          | Build all 6 library packages                |
| `pnpm test:ci`             | Run all tests with coverage                 |
| `pnpm lint`                | Lint all projects                           |

## Architecture

```
ng-forge/
├── packages/
│   ├── dynamic-forms/               # Core library (@ng-forge/dynamic-forms)
│   ├── dynamic-forms-material/      # Material UI adapter
│   ├── dynamic-forms-bootstrap/     # Bootstrap UI adapter
│   ├── dynamic-forms-primeng/       # PrimeNG UI adapter
│   ├── dynamic-forms-ionic/         # Ionic UI adapter
│   └── dynamic-form-mcp/           # MCP server for AI-assisted form generation
├── apps/
│   ├── docs/                        # Documentation app (SSR with Analog)
│   └── examples/
│       └── sandbox/                 # Unified example app (all 4 adapters + E2E specs)
├── internal/
│   ├── sandbox-harness/             # SandboxHarness, SandboxMountDirective, SANDBOX_FORM_CONFIG
│   ├── sandbox-adapter-{material,bootstrap,primeng,ionic}/ # Adapter factory functions
│   ├── examples-{material,bootstrap,primeng,ionic}/ # Field-type scenario components
│   └── examples-shared-ui/          # ExampleScenarioComponent
├── .claude/skills/                  # Custom Claude Code skills
└── scripts/                         # CI/Docker/deployment helpers
```

**Dependency direction:** UI adapters depend on core. Core has zero knowledge of adapters. Adapters provide field components + mappers that plug into core's registry via `provideDynamicForm(...withMaterialFields())`.

**Data flow:** `FormConfig` → `FormStateManager` (state machine + field resolution) → `ResolvedField[]` (component + injector + inputs signal) → `NgComponentOutlet` renders each field.

## Environment

- **Node.js**: `>=24.0.0`
- **pnpm**: `>=10.0.0`
- **Angular**: `~21.1.0`
- **TypeScript**: `~5.9.2`
- **Nx**: `22.4.5`
- **Docs app**: AnalogJS (file-based routing, SSR pre-rendering). Route components use `default` exports. Be aware of Vite/Nx cache issues — suggest cache clearing (`nx reset`) when encountering ghost errors after config changes


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ng-forge/ng-forge](https://github.com/ng-forge/ng-forge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
