---
trigger: always_on
description: 1. **Commit messages MUST be in English.** Never use Japanese or any other language in commit messages.
---

# megane - Claude Code Instructions

## CRITICAL RULES

1. **Commit messages MUST be in English.** Never use Japanese or any other language in commit messages.
2. **NEVER use Puppeteer.** The repo uses **Playwright**:
   - The current `*.spec.ts` E2E suite uses `@playwright/test` via the local `playwright` 1.56 devDependency (installed by `npm ci`).
   - Legacy `*.mjs` scripts (`scripts/dev-preview.mjs`, `scripts/capture-screenshots.mjs`, `tests/e2e/test_vscode_render.mjs`, `tests/e2e/vscode_full_screen.test.mjs`, etc.) resolve Playwright from the global install via `createRequire("/opt/node22/lib/node_modules/")`. The shared helper is `tests/e2e/utils/playwright.mjs` — reuse it instead of duplicating the `createRequire` block.
   - `puppeteer` is **not** a dependency. Do not add it.
3. **Always build WASM before running the dev server or full build.** The WASM pkg directory (`crates/megane-wasm/pkg/`) does not exist until `npm run build:wasm` is run. In sandboxes the `wasm-opt` download is blocked, but `npm run build:wasm` now self-heals via `scripts/build-wasm.mjs` (auto-retries `wasm-pack --no-opt` so `pkg/package.json` is always written) — do **not** hand-edit `Cargo.toml` to toggle `wasm-opt` anymore. Force the fast path with `MEGANE_WASM_NO_OPT=1`. See the `build` skill.
4. **Always create a PR after pushing changes.** Use `gh pr create` to open a pull request. PR titles and descriptions must be in English. See the `github-cli` skill for remote URL workaround. Before reporting completion, verify CI passes with `gh run list`.
5. **In plan mode, strictly follow the approved plan.** Do not skip steps, reorder them, or add unplanned work. If the plan needs changes, explain the reason and get approval before deviating.
6. **All file formats should behave consistently across hosts.** When you add a parser or feature to one platform (standalone webapp, Jupyter widget, JupyterLab labextension, VSCode extension), register it on every host unless there is a host-specific reason not to. The single source of truth is `docs/docs/platform-support.md`; update its tables in the same PR. Host registration points: `crates/megane-wasm/src/lib.rs` (browser parsers), `src/components/nodes/LoadStructureNode.tsx` / `LoadTrajectoryNode.tsx` (standalone accept lists), `jupyterlab-megane/src/filetypes.ts`, `vscode-megane/package.json` `customEditors`. Walk the full checklist in the `add-format` skill — drift between hosts is the #1 source of "format X works in the webapp but not in VSCode/JupyterLab" bugs.
7. **The Jupyter widget (anywidget `MolecularViewer`) does not mount the visual pipeline editor.** Pipeline data still flows in via `MolecularViewer.set_pipeline()` (`_pipeline_json` + `_node_snapshots_data`), but the in-cell `PipelineEditor` UI is intentionally not rendered — the host cell chrome cannot reliably lay it out. Do not re-introduce a `pipeline=True` opt-in or a `_pipeline_enabled` traitlet. Visual editing lives in the standalone webapp, JupyterLab labextension, and VSCode extension only.
8. **Codecov is a hard merge gate — write tests for every new line you add.** The `test-rust`, `test-ts`, and `test-python` jobs in `.github/workflows/ci.yml` upload coverage to Codecov with `fail_ci_on_error: true`, and `codecov.yml` requires **patch coverage ≥ 70 %** on every PR (project coverage is off, only the diff is gated). New parsers, pipeline nodes, React components, Python API, and Rust modules MUST ship with unit tests in the same PR — relying on E2E does not count because E2E coverage is unmeasured by Codecov. Reproduce the gate locally before pushing: `npm test -- --coverage` (TS → `coverage/ts/lcov.info`), `cargo llvm-cov --package megane-core --lcov --output-path lcov.info` (Rust), `python -m pytest --cov-report=xml:coverage.xml` (Python). The `make coverage-all` target (or `make coverage-ts` / `coverage` / `coverage-rust`) wraps these. If you genuinely cannot cover a line (e.g. unreachable defensive branch) document why in the PR description rather than disabling the check. See the `testing` skill for details.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [megane-labs/megane](https://github.com/megane-labs/megane) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
