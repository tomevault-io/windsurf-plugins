---
trigger: always_on
description: This repository is built around agent-driven Chrome DevTools Protocol (CDP) control for Avalonia applications. Both the sample target app and the inspector app expose CDP endpoints, so agents can inspect, control, record, replay, and verify UI behavior without platform-specific desktop automation.
---

# Agent Guide: CDP Avalonia Self-Inspection and Testing

This repository is built around agent-driven Chrome DevTools Protocol (CDP) control for Avalonia applications. Both the sample target app and the inspector app expose CDP endpoints, so agents can inspect, control, record, replay, and verify UI behavior without platform-specific desktop automation.

## Core Architecture

- `CdpSampleApp` is the target application. It starts CDP on `http://127.0.0.1:9222`.
- `CdpInspectorApp` is both an inspector client and a CDP target. It starts CDP on `http://127.0.0.1:9223`.
- Agents use `/json` or `/json/list` to discover each WebSocket endpoint.
- Agents drive UI through CDP domains such as `DOM`, `Input`, `Runtime`, `Page`, `Accessibility`, and `Recorder`.
- The inspector can connect to the sample and then control the sample through its preview pane, Test Studio, recorder, and view models.

Typical dual-CDP topology:

```text
Agent process or script
  |-- CDP WebSocket 9223 --> CdpInspectorApp
  |-- CDP WebSocket 9222 --> CdpSampleApp

CdpInspectorApp
  |-- internal CDP client connection --> CdpSampleApp
```

## When To Use Each Verification Mode

Use the mode that matches the task. Do not force every task through the same scenario.

- For code changes that affect CDP, inspector behavior, recording, replay, visual tree traversal, or sample interactions, create a task-specific verifier with explicit assertions.
- **Mandatory E2E Preview Simulation**: For all implemented code changes, performing a preview-based E2E recording and replay verification (simulating user interactions via the preview canvas `#imgScreenshot` inside the inspector and replaying them with 0 failures) is **strictly mandatory**.
- For normal feature work, `scratch/ControlApp` or `scratch/InteractiveDemo` may be used as a dynamic test harness, but the scenario must be custom to the change being verified.
- For live demos or requests that explicitly say not to use `scratch/ControlApp`, launch the real apps and drive them directly through their CDP ports.
- Do not reuse a static recording/replay script unless the user explicitly asks for that exact static flow.
- Always report the exact verification evidence: commands run, endpoints used, step counts, pass/fail counts, generated report paths, frame counts, screenshots, or relevant runtime state.

## E2E YAML Test Suite Requirements

All new inspector features, tabs, or major code changes must be covered by structured E2E YAML test files inside `tests/CdpInspectorApp.E2e/`.

### Directory Structure & Categorization
The test suite is organized into subfolders matching the feature areas:
- `tests/CdpInspectorApp.E2e/connection/` (connection flows)
- `tests/CdpInspectorApp.E2e/simulation/` (preview simulation)
- `tests/CdpInspectorApp.E2e/elements/` (visual tree & styles panel)
- `tests/CdpInspectorApp.E2e/console/` (eval scripts & REPL)
- `tests/CdpInspectorApp.E2e/sources/` (workspace files explorer)
- `tests/CdpInspectorApp.E2e/network/` (outbound HTTP request capture)
- `tests/CdpInspectorApp.E2e/performance/` (performance chart metrics)
- `tests/CdpInspectorApp.E2e/profiler/` (profiling capture runs)
- `tests/CdpInspectorApp.E2e/memory/` (memory control allocations)
- `tests/CdpInspectorApp.E2e/recorder/` (test recorder & synchronizer)

Each individual test case must be stored in its own separate `.flow.yaml` file (e.g. `connect.flow.yaml`).

### YAML Flow Structure & Metadata Separator
YAML files must be split into two documents using the standard `---` document separator:
1. **Metadata Document**: Contains `appId: "CdpInspectorApp"`, `description`, `tags`, and optionally `env`.
2. **Steps Sequence Document**: A list of sequential execution step mappings using action names as keys (e.g., `tapOn`, `delay`, `evalScript`, `assertTrue`).

Example YAML test file:
```yaml
appId: "CdpInspectorApp"
description: "Tests element view visual tree traversal"
---
- delay: "1000"
- tapOn: "#TabElements"
- delay: "1000"
- assertTrue: "document.querySelector('#TabElements') != null"
```

### Replay & Verification Runner
Verify all test suites using the `cdp-cli run` command pointing to the suite directory:
```bash
dotnet run --project src/CDP.Inspector.CLI/CDP.Inspector.CLI.csproj -- -p 9223 run tests/CdpInspectorApp.E2e/ --report --video
```

### Reusable Sub-Flow Library
To avoid redundant code and speed up authoring, common UI actions are structured as sub-flows inside `tests/CdpInspectorApp.E2e/shared/`. Invoke them in any main flow file via the `runFlow` action:
```yaml
- runFlow: "shared/connect_to_sample.yaml"
- runFlow: "shared/navigate_to_profiler.yaml"
```

### Playwright Code Generation & Headless CI
E2E testing requires exporting YAML flows to executable Playwright scripts for automated headless execution inside CI/CD pipelines.
1. **Generate Playwright Scripts**: Use the CLI `codegen` tool:
   ```bash
   dotnet run --project src/CDP.Inspector.CLI/CDP.Inspector.CLI.csproj -- codegen tests/CdpInspectorApp.E2e/ --playwright-out tests/playwright/
   ```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wieslawsoltes/CDP](https://github.com/wieslawsoltes/CDP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
