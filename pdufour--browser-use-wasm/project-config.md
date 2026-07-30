---
trigger: always_on
description: E2E only — one Playwright spec, no unit tests
---


# E2E only (no unit tests)

```bash
npm run cache:showui && npm run test
```

## The only automated tests

| Allowed | Forbidden |
|---------|-----------|
| `tests/e2e/e2e.spec.js` — **34** gate cases on one model + optional **benchmark** block (`E2E_BENCHMARK=1`) | Unit tests (`vitest`, `jest`, `node:test`, `tests/unit/`, `*.test.js`, `*.spec.js` outside `e2e/`) |
| `tests/e2e/e2e.js` — blackbox helpers (no `import` from `src/`) | Extra Playwright spec **files** or `testMatch` broadening |
| `tests/e2e/global-setup.js`, `tests/e2e/fixtures/` | `MULTI_MODEL`, env matrices, or second E2E suites in CI |
| `npm run test` → `playwright test` | `test:multi`, `test:all`, or scripts that run additional Playwright tests |

**Green-circle workflow:** autoload → capture (automatic in the UI; E2E drives the off-screen shelf `#btn-capture`) → Goal `click Submit` → **Run task** → green button at the marker on the screenshot.

**Gate model:** `E2E_MODEL` env (default `ShowUI-2B`) — must match `BROWSER_VALIDATED_MODEL_IDS` in `src/config/models/registry.ts`. CI runs this model only. Compare all models with `npm run test:benchmark`, not default `npm run test`.

**Smoke:** autoload `data-model-loaded` + ShowUI switcher; empty `#prompt` disables Run task.

**Model switcher (blackbox `#model-switcher`, `#model-status` dataset only):**

- **Uncached** pick reverts picker (`E2E_UNCACHED_MODEL`, default `GUI-G2-3B` when not in manifest).
- **Switch round-trip** (`ShowUI-2B` ↔ `E2E_SWITCH_MODEL`, default `MAI-UI-2B`) — **benchmark only** (`npm run test:benchmark`), not the default gate suite.

**Run task (UI):** 3× `click Submit` green band; `click Sign in` outside Submit band; **no marker until task**; `click Cancel` not green + far from Submit; **second task** on same capture; **prompt Enter submits** the goal form (same asserts as button); **`type paul in the email field`** → parsed INPUT action + live email value (no DOM coords); **address-bar navigation** (`?e2enav=1`) → re-capture → Submit task still grounds; **multi-step modal popup flow** — ONE instruction opens the Help dialog, types into its Contact email field, then closes it (agent loop, `MULTI_STEP_TASK_WAIT_MS` composed from existing ceilings, end-state asserts only).

**UI (no mic):** Help button opens/closes modal; **Cmd/Ctrl+Shift+S** flips `body[data-viewport]` live ↔ snapshot (pure DOM).

**Voice workflows (`?e2e=1`, `__e2eVoiceTool` structured tool calls — no mic, no phrase regex in product):**

- **Fake cursor:** hover Submit / move Cancel → marker + green at Submit, Cancel distance ≥ 0.08.
- **Click / bare Submit:** `click Submit` or `Submit` → grounded on screenshot.
- **Hover Cancel:** fake cursor visible, transcript mentions Cancel.
- **Capture page:** `capture page` → `captureGeneration` increases + `#screenshot-img`.
- **Scroll down / scroll to top:** live scroll + re-capture or `scrollTop === 0`.
- **Select country:** `select Canada in Country` → `#checkout-country` value.
- **Press Tab:** Tab success → capture generation increases.
- **Modal Escape:** `open modal` → modal visible → `press Escape` → hidden.
- **Toggle Remember me:** checkbox flips.
- **Type email:** live input value `e2e@test.com`.
- **Focus / blur / clear email:** vision-grounded (ShowUI point → element at point) focus, blur, empty value.
- **Scroll to top:** scrollable main region `scrollTop === 0` (blackbox via voice/capture).

No coords from hook return values or live DOM button layout.

`playwright.config.js` must keep `testMatch: '**/e2e.spec.js'` only.

## Model benchmark (opt-in)

```bash
npm run cache:public && npm run test:benchmark
```

- `E2E_BENCHMARK=1` — **off** in default CI (`npm run test`). `E2E_EXPENSIVE=1` is a deprecated alias.
- Results default to `e2e-benchmark-results.txt`; gate uses `e2e-results.txt` — both gitignored (scripts must not dirty the repo).
- One case per `E2E_BROWSER_LOADABLE_MODEL_IDS` entry in manifest + switch round-trip.
- Per model: `openE2eSessionForModel` → **Load** → capture → `click Submit` task (`INFERENCE_TIMEOUT_MS` unchanged).
- `E2E_BENCHMARK_MODELS=id1,id2` narrows the list.

## Objective timeouts (non-negotiable — keep low)

These are the **real** failure ceilings in `tests/e2e/e2e.js`. If a case hits them, fix WebGPU/worker/capture — **do not** raise the constant or add retry time.

| Constant | ms | Use |
|----------|-----|-----|
| `INFERENCE_TIMEOUT_MS` | **12_000** | Run task / `waitForParsedTask` — must match `INFERENCE_TIMEOUT_MS` in `src/config/vl.ts` |
| `VOICE_GROUNDING_WAIT_MS` | **12_300** | Voice tools that call ShowUI (pointer tools `click`/`hover`/…, `input`, `select`, `toggle_checkbox`, `clear_field`, `focus_field`, `blur_field`, fake cursor) |
| `CAPTURE_READY_TIMEOUT_MS` | **20_000** | Capture + JPEG encode ready (`dataset.captureReady`) |
| `VOICE_DOM_WAIT_MS` | **5_000** | Voice DOM-only tools (modal, Tab, scroll, scroll_to_top) |
| `LOAD_TIMEOUT_MS` | **60_000** | `openE2eSession` autoload ShowUI-2B only |

| `SCREENSHOT_READY_TIMEOUT_MS` | **10_000** | `#screenshot-img` after SnapDOM (never unbounded `waitForFunction`) |
| `E2E_TEST_TIMEOUT_MS` | **120_000** | Playwright per-test ceiling (`e2e.spec.js` + `playwright.config.js`) — autoload envelope only |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pdufour/browser-use-wasm](https://github.com/pdufour/browser-use-wasm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
