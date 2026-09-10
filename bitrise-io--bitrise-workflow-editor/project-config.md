---
trigger: always_on
description: How to write code here. Mechanism lives in [flows.md](flows.md), rationale in
---

# Conventions

How to write code here. Mechanism lives in [flows.md](flows.md), rationale in
[decisions.md](decisions.md), vocabulary in [domain.md](domain.md).

## What you are writing with

| | |
|---|---|
| Framework | React 18 + TypeScript (strict), built with Vite |
| UI | `@bitrise/bitkit-v2` (Chakra v3) for all new work. `@bitrise/bitkit` (Chakra v2) is legacy; migrate v1 components to v2 in any file you touch |
| State | Zustand, `BitriseYmlStore` |
| Data fetching | TanStack React Query |
| Routing | wouter, lazy-loaded pages |
| YAML editing | Monaco + monaco-yaml + `@bitrise/languageserver` |
| Graphs | XYFlow + dagre |
| Drag and drop | dnd-kit |
| Path alias | `@/` maps to `source/javascripts/` |

## Where things live

```
source/javascripts/
  core/            no React (lint-enforced), no DOM (convention)
    models/        internal types
    api/           HTTP clients: DTO in, model out. Reach them through a hook
    services/      business logic over models. All structured YAML mutation
    stores/        Zustand stores, mainly BitriseYmlStore
    utils/         YmlUtils and friends
  hooks/           store selectors and React Query wrappers
  components/      shared and unified-editor
  pages/           thin composition

apiserver/         Go HTTP server (Gorilla Mux), API plus embedded assets
cmd/               Go CLI (Cobra)
spec/              Jest unit and Playwright E2E
```

### Adding a field users can edit

The order matters, because each layer depends on the one before it.

1. **`core/models/BitriseYml.ts`** — add the key to the matching model, for example
   `WorkflowModel`. The rest of the stack is typed off these.
2. **`core/services`** — a setter through `updateBitriseYmlDocument`, plus a validator if the
   value can be wrong.
3. **`hooks/`** — a selector hook if the UI needs to read it back.
4. **The component**, in `components/` or a page.
5. **A `*.spec.ts` beside the service**, using the round-trip idiom under [Testing](#testing).

The gate that is not in this repo: the Go server validates the saved config with the `bitrise`
CLI, so a key the CLI does not know fails at save even though every layer above compiled. Check
the key exists in the CLI's schema before building UI on it. The call is
`apiserver/utility/utility.go`, which hands the config to `CreateBitriseConfigFromCLIParams` under
`ValidationTypeFull`. That is the `bitrise` library pinned in `go.mod`, compiled into the server —
not whatever CLI the user has installed, which `bitrise-plugin.yml` only floors at a minimum
version. So the schema you are checking against moves when someone bumps that dependency.

For a modular config the server validates the **merged** YAML, not each file. A module that is
invalid standing alone is fine, and the thing that has to parse and validate is the merge.

### Where does this go?

| You are writing | It goes in |
|---|---|
| Logic that changes the YAML structure | `core/services` |
| A name or value validator | `core/services`, returning `string \| boolean` |
| An HTTP call | `core/api`, wrapped by a hook |
| Reading YAML state into a component | a selector hook over `yml` |
| Fetching remote data | a React Query hook |
| Which dialog is open | the page store, or `useDisclosure` on simple pages |
| A mode difference | `core/api` or the component. Never a service |
| Anything touching `yaml` AST nodes | nowhere directly. Use `YmlUtils` |

## Layer rules

**`core/` is framework-agnostic.** No React, no DOM, so services test in plain Jest with no
renderer. Lint enforces only the React half, by banning the `react` and `react-dom` imports.
Nothing stops you reaching for `document` or `window`, and `CommonUtils`, `WindowUtils` and
`PageProps` already do. Anything that needs React goes in `hooks/`, anything that needs the DOM
goes in `components/`.

**Dependency direction.** Most services import no other service at all. The exceptions are few
enough to list: `WorkflowService` reaches for `EntityIndexService`, `ContainerService` and
`PipelineService` for two each, and `BitriseYmlService`, `EnvVarService`, `StackAndMachineService`
and `ToolsService` for one. `StepService` imports none. Check the imports before assuming a layer.

`MODE` is set by the npm script, not by config: `npm start` exports `MODE=CLI`,
`npm run start:website` exports `MODE=WEBSITE`, and Vite passes it through `envPrefix` into
`window.env`, which is why it does not exist under Jest.

**Branch on runtime mode at the edges only.** `MODE=CLI` is the plugin default, `MODE=WEBSITE` is
the monolith iframe. Branch in `core/api` for endpoint paths and request shapes, in components for
feature visibility, in `core/analytics`. Nothing enforces this: wanting `isWebsiteMode()` inside a
service means the branch belongs somewhere else, but no lint rule will tell you. The one standing
exception is `BitriseYmlStore.warnInDev`, which asks `RuntimeUtils` whether it is in production to
decide whether to warn, not to change behaviour. Mode is
sometimes a capability difference rather than a URL swap: `SecretApi.getSecretValue` returns
`undefined` in CLI mode because no local endpoint exists.

## Services

Pure functions exported through one `export default { … }`, never classes. Pure in the sense

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bitrise-io/bitrise-workflow-editor](https://github.com/bitrise-io/bitrise-workflow-editor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->
