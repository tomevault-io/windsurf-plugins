---
trigger: always_on
description: This repo holds many packages. `packages/coding-agent/` is the subject of a request unless it names
---

# Development Rules

## Default context

This repo holds many packages. `packages/coding-agent/` is the subject of a request unless it names
another one. "Agent" in a request means that CLI, not the assistant answering.

Maps and indexes: [`packages/coding-agent/DEVELOPMENT.md`](packages/coding-agent/DEVELOPMENT.md) maps
the source tree to its owning documents; [`docs/internal/README.md`](docs/internal/README.md) indexes
contributor docs ([onboarding](docs/internal/onboarding.md), [testing](docs/internal/testing.md));
[`review.md`](review.md) is the pull-request review guide; [`docs/handbook/`](docs/handbook/) is the
operator manual.

|Member|Description|
|---|---|
|`contracts/wire`|Wire and presentation types, so a browser, a test client or a second host need not depend on coding-agent; every block, stop reason and usage a guest reads is a projection of `@veyyon/model`, imported type-only|
|`contracts/view`|Dependency-free tool view model, so a tool describes its output without constructing a terminal component|
|`contracts/settings`|Dependency-free setting declaration vocabulary, so a package declares a setting without importing the store that persists it or the host that draws it|
|`contracts/model`|Dependency-free model and message vocabulary, so a provider plugin implements a stream and a host reads a turn without importing the catalog that resolves the model or the client that drives it|
|`contracts/session`|What a session file is made of: the entry vocabulary, the message union it records and the hooks a package augments to add its own, so the agent that writes a file, the compaction that rewrites it and the host that displays it read one definition|
|`contracts/host`|What a host offers a plugin, each capability optional and absent when the host cannot honour it, so a tool asks for a notification without naming the terminal or the window that delivers it|
|`contracts/tool`|What a tool declares and returns, so a tool states its spec, approval tier and result without importing the loop that schedules it or the host that prompts for approval; imports `@veyyon/model` type-only|
|`contracts/plugin`|Dependency-free plugin manifest vocabulary, so a package declares what its `package.json` `veyyon` field contributes without importing the loader that installs it|
|`kernel`|The only member that is not a plugin — the plugin loader, the contribution registry and the session spine, naming no tool and no host|
|`packages/ai`|Multi-provider LLM client with streaming support|
|`packages/catalog`|Model catalog: bundled models.json, provider descriptors, model identity/classification|
|`packages/agent`|Agent runtime with tool calling and state management|
|`packages/coding-agent`|Main CLI application (primary focus)|
|`hosts/gui`|Graphical host: draws the same `ToolView` models as HTML, the second implementation that keeps `contracts/view` a contract rather than a description of the terminal|
|`hosts/terminal/engine`|Terminal UI library with differential rendering|
|`apps/stats`|Local observability dashboard (`veyyon stats`)|
|`packages/utils`|Shared utilities (logger, streams, temp files)|
|`plugins/argot`|Per-project shorthand vocabularies: lossless substitution codec over `AGENTS.dict`. Published standalone — depends on nothing in this repo|
|`plugins/hashline`|Line-anchored patch language the edit tool applies, with a pluggable filesystem backend|
|`plugins/mnemopi`|Local SQLite memory engine: triples, embeddings, recall|
|`packages/tool-render`|Shared React tool-call renderers for HTML export, collab-web and the stats dashboard|
|`clients/web`|Browser guest client and local relay for collab live sessions (private)|
|`plugins/mode-swarm`|Swarm orchestration extension|
|`plugins/web`|Site scrapers that turn a URL into markdown: ~80 per-site handlers, the page loader escalation ladder and the Parallel extract client, running against host capabilities passed in rather than imported|
|`tests/evals`|Every model and agent evaluation: the DeepSWE, Terminal-Bench 3.0 and TypeScript-edit suites, harness adapters, execution backends, run store, REST/SSE API and live dashboard (private)|
|`tests/simulations`|Deterministic offline simulations driving real subsystems end to end (private)|
|`natives/bridge/addon`|The napi addon: the only Rust surface TypeScript calls (grep, glob, text measurement, highlighting, clipboard, SIXEL)|
|`natives/bridge/bindings`|Bindings for native text/image/grep operations|
|`natives/code/ast`|Structural search, replace and code-block summaries over tree-sitter and ast-grep|
|`natives/diff/kernel`|Line-comparison engine for unified diff, ported from GNU diff `compareseq` and `shift_boundaries`|
|`natives/diff/uu-diff`|`diff` as an in-process shell builtin|
|`natives/fs/iso`|Copy-on-write filesystem isolation and change diffing (APFS clonefile, Linux overlayfs, Windows ProjFS)|
|`natives/fs/uutils-ctx`|Thread-local stdio and cwd the uutils builtins run against|
|`natives/search/glob`|Glob normalization, brace expansion, depth bounds and compilation|
|`natives/search/grep-kernel`|One compiled matcher over regex and PCRE2 for every search path|
|`natives/search/uu-grep`|`grep` as an in-process shell builtin, ripgrep-backed|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [santhreal/veyyon](https://github.com/santhreal/veyyon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
