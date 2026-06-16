---
trigger: always_on
description: This file describes the codebase conventions and invariants that AI agents must follow when contributing to this project.
---

# Agent Guidelines

This file describes the codebase conventions and invariants that AI agents must follow when contributing to this project.

## Commands

```bash
npm run dev                # start dev server (http://localhost:5173)
npm run build              # TypeScript check + Vite production build
npm run typecheck          # tsc -b (no emit) — fast type-only check
npm run lint               # ESLint
npm run lint:fix           # ESLint with --fix
npm run test:unit          # Vitest — fast, no browser required
npm run test:unit:coverage # Vitest with v8 coverage (./coverage)
npm run test:e2e           # Playwright — requires dev server or build
npm test                   # both test suites
```

Always run `npm run test:unit` after changing anything in `src/lib/` or `src/hooks/`. Run `npm run test:e2e` after changing UI components. Run `npm run lint` AND `npm run typecheck` before finishing.

A pre-commit hook (`.husky/pre-commit`) runs `lint-staged` (eslint --fix on staged files), `tsc -b`, and the unit suite. Do not bypass with `--no-verify`.

## Architecture

The project is a pure client-side React app. No backend, no API routes.

**Core data flow:**
1. User selects model/quant/context in `ConfigCard`
2. `useCalcResult` (`src/hooks/`) calls `calcLLMRam()` → RAM breakdown
3. `calcDisk()` → storage breakdown
4. `useValueScore` (`src/hooks/`) calls `calcValueScore()` → TPS and cost-efficiency (requires hosting data)
5. Results render in `ResultCard` and `AvailableHardware`
6. State serialized to URL via `encodeState`/`decodeState` (500ms debounce)

## Key Invariants — Do Not Break

### KV cache formulas (`src/lib/calculator.ts`)

There are 4 KV formulas. Each must stay consistent across `calcLLMRam` and `calcValueScore` — both functions implement the same switch. If you change a formula in one, change it in the other.

| Formula | Used by |
|---|---|
| `standard` | Llama, Qwen, Mistral, Phi |
| `hybrid` | Gemma 2/3, Mistral Sliding Window |
| `mla` | DeepSeek V3/R1 |
| `linear_hybrid` | Qwen 3.5 |

### q1 weight overhead factor

```ts
const weightOverhead = quant === "q1" ? 1.0 : 1.1;
```

For Q1 (1-bit / MLX format) the overhead is already baked into `QUANT_BITS["q1"] = 1.25`. All other quants use 1.1 to account for embeddings and norms stored in higher precision. Do not remove this distinction.

### QUANT_BITS vs QUANT_BYTES

`QUANT_BITS` (in `quants.ts`) is used for RAM/disk calculations and works in integer-like bits.
`QUANT_BYTES` (in `calculator.ts`) is used for TPS/bandwidth calculations and uses fractional bytes per parameter.
They are separate for a reason — do not merge them. They MUST stay in sync though:
`QUANT_BYTES[q]` ≈ `QUANT_BITS[q] / 8` for every `q`. There is a parameterised
test in `calculator.test.ts` ("QUANT_BYTES matches QUANT_BITS / 8") that
will fail loudly if drift sneaks in.

### Quantization families & engine compatibility

`QUANT_SPECS` in `quants.ts` is the single source of truth for every quant we
support. Each entry carries a `family` field (`float | gguf | gptq | awq | mlx`)
which drives two things:

1. UI grouping in the Weights Quant dropdown (`getWeightQuantGroups`).
2. Inference-engine filtering (`QUANT_FAMILY_ENGINES`) — picking GPTQ/AWQ
   hides `llamacpp`; picking GGUF/MLX hides `vllm` / `tensorrt`. The full
   matrix lives in `QUANT_FAMILY_ENGINES` and `"custom"` is universally
   compatible (escape hatch for niche runtimes).

When adding a new quant: update `QUANT_SPECS` AND `QUANT_BYTES` (calculator.ts)
in one PR, otherwise the bits/bytes invariant test will fail. If the quant
introduces a new family, also extend `QUANT_FAMILY_ENGINES` and the family
labels documented in `QuantSelector.tsx` / `Footer.tsx`.

Effective bpw rules of thumb (see comments on `QuantSpec.bpw`):
- GPTQ g128 asym: bits + 0.25 (FP16 scale + zero point amortised over 128)
- AWQ g128:       bits + 0.25 (FP16 scale + scaled zero)
- MLX g64:        bits + 0.5  (FP16 scale + bias, smaller groups → more overhead)
- GGUF Q*_K_M:    bits exactly (block scales already counted in upstream specs)
- Q1 (sign-bit):  1.25 bpw (scale baked into QUANT_BITS — see q1 overhead note)

### URL state encoding

`encodeState`/`decodeState` in `src/lib/state.ts` use UTF-8 → base64url (no `+`, `/`, `=`). The encoding must remain stable — any change that breaks decoding of existing URLs is a breaking change for shared links.

### parseHfUrl

The regex must stop at `?`, `#`, and whitespace. Do not simplify it to `[^/]+` — that would include query params in the repo ID and cause 404s on HF API calls.

### `engineId` ⇄ `kvCacheFillPct` synchronisation

`ModelSettings.engineId` (a stable string id like `"llamacpp"` / `"vllm"` / `"tensorrt"` / `"custom"`) and `ModelSettings.kvCacheFillPct` (the numeric value used by `calcLLMRam` / `calcValueScore`) must stay in sync. The parent (`ConfigCard`) is responsible:

- selecting an engine preset → update **both** `engineId` AND `kvCacheFillPct` in one `setState`
- typing in the manual % input → update `kvCacheFillPct` AND stamp `engineId: "custom"`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [smelukov/WeightRoom](https://github.com/smelukov/WeightRoom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
