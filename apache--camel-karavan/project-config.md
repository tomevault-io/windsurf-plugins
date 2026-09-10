---
trigger: always_on
description: Operating guide for an AI agent working on the **Karavan VS Code extension**.
---

# AGENT.md — Apache Camel Karavan VS Code Developer Agent

Operating guide for an AI agent working on the **Karavan VS Code extension**.
Everything below was verified against the code in this checkout at extension version
**4.22.0**. Paths are relative to `karavan-vscode/` unless stated otherwise.

---

## 1. What this project is

Karavan is an *Integration Toolkit for Apache Camel*. The VS Code extension
(`karavan-vscode`, publisher `camel-karavan`, marketplace id `camel-karavan.karavan`)
embeds the Karavan visual designer as a **webview** and drives Camel JBang / Maven from
the VS Code UI.

It reads and writes **Camel YAML**: plain routes (`*.camel.yaml`), Kamelets
(`*.kamelet.yaml`) and Integration CRDs (`kind: Integration`).

### Layout

```
/karavan
├── docs/                    ← user + developer docs (DEV.md, VSCODE_HOWTO.md, …)
└── karavan-vscode/          ← the extension (the only module checked out here)
    ├── AGENT.md             ← this file
    ├── src/                 ← extension host code (Node, webpack target "node")
    ├── webview/             ← React UI bundled into dist/webview.js
    │   ├── index.tsx App.tsx vscode.ts prerender.tsx
    │   ├── karavan/         ← designer, topology, documentation, stores
    │   └── karavan-core/    ← vendored copy of karavan-core (Camel model + YAML API)
    ├── metadata/            ← components.json (7.3M), kamelets.yaml (1.9M), spiBeans.json
    ├── snippets/            ← Java templates per runtime (Processor, AggregationStrategy)
    ├── icons/ images/
    ├── webpack.config.js tsconfig.json package.json
    └── dist/                ← build output (gitignored)
```

> Other monorepo modules (`karavan-app`, `karavan-core`, `karavan-designer`,
> `karavan-generator`, `karavan-devmode`, `release-utils`, `.github/**`) are *OUT OF SCOPE*. Do not read them; 
> The repository root `/karavan` is not writable; 
> write inside `karavan-vscode/` or `docs/`.

---

## 2. Architecture

### 2.1 Three bundles, one build

`webpack.config.js` exports three configs, all built in one run:

| Config       | Entry                   | Target  | Output |
|--------------|-------------------------|---------|--------|
| extension    | `src/extension.ts`      | node    | `dist/extension.js` (commonjs2, `vscode` external) |
| webview      | `webview/index.tsx`     | web     | `dist/webview.js` + `dist/main.css` |
| prerender    | `webview/prerender.tsx` | node    | `dist/prerender.js` (SSR helper; excluded from the VSIX by `.vscodeignore`) |

`package.json` declares both `main` and `browser` as `./dist/extension.js`, so the
extension is also meant to run as a **web extension** (`vscode-test-web`,
`--extensionDevelopmentKind=web`). Keep `src/` free of Node-only APIs — file I/O already
goes through `workspace.fs`, not `node:fs`.

### 2.2 Extension host (`src/`, ~1.9k LOC)

| File | Responsibility |
|------|----------------|
| `extension.ts` | `activate()` — registers every command and tree view; `exportAndRunProject()`; `deactivate()` clears the `karavan:loaded` context key. |
| `designerView.ts` | Owns designer webview panels (`KARAVAN_PANELS: Map<relativePath, WebviewPanel>`), the message protocol, and data loading. |
| `topologyView.ts` | Single `topology` panel showing the route topology graph. |
| `helpView.ts` | "Help & Feedback" tree + the Knowledgebase webview. |
| `integrationView.ts` | `integrations` tree: scans `*.yaml`, keeps those `CamelDefinitionYaml.yamlIsIntegration()` accepts; children are the flows. |
| `openapiView.ts` | OpenAPI discovery (`*.json` / `*.yaml` with an `openapi` key) + the "Generate REST API" quick-pick flow. |
| `jbang.ts` | Builds Camel JBang command lines (`run`, `export`, `generate rest`). |
| `exec.ts` | `shelljs` exec + a `Map<id, Terminal>` of reusable terminals; `runWithRuntime()`, `camelDeploy()`. |
| `maven.ts` | Two command builders — **currently unreferenced**. |
| `utils.ts` | Everything else: file I/O via `workspace.fs`, kamelet/component/template/bean/java loading, `application.properties` handling, naming rules. |
| `webviewContent.ts` | The HTML shell for every panel (loads `dist/main.css` + `dist/webview.js`). |

### 2.3 Webview (`webview/`)

- `index.tsx` mounts `<ThemeProvider><App/></ThemeProvider>` into `#root`.
- `App.tsx` is a **class component** holding all state and is the sole message endpoint.
  It renders one of three pages: `designer` (`KaravanDesigner`), `topology`
  (`TopologyTab`) or `knowledgebase` (`DocumentationPage`).
- `webview/vscode.ts` exports the `acquireVsCodeApi()` handle (`undefined` outside VS
  Code — the prerender build relies on that).
- State: **zustand** stores (`stores/*.ts`, `designer/DesignerStore.ts`,
  `designer/CodeStore.ts`), created with `createWithEqualityFn` + `shallow`.
- Cross-component events: **rxjs** `Subject`s in `designer/utils/EventBus.ts`
  (`DslPosition`, `Command`, `IntegrationUpdate`, alerts).
- Host callbacks are injected into the designer through the static
  `designer/utils/InfrastructureAPI` class (`setOnSave`, `setOnSaveCustomCode`,
  `setOnInternalConsumerClick`, …). `KaravanDesigner` wires these in its mount effect.
- UI kit: PatternFly 6 (`react-core`, `react-topology`, `react-table`), Monaco editor,
  `elkjs` for topology layout.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apache/camel-karavan](https://github.com/apache/camel-karavan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->
