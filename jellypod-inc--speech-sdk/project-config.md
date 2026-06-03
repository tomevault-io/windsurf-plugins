---
trigger: always_on
description: This file provides guidance when working with code in this repository.
---

# Speech SDK

This file provides guidance when working with code in this repository.

## Commands

```bash
pnpm install              # install dependencies
pnpm build                # compile TypeScript (tsc)
pnpm test                 # run unit tests
pnpm test -- -t "test name"  # run a single test by name
pnpm run test:e2e         # run e2e tests (requires OPENAI_API_KEY / ELEVENLABS_API_KEY)
pnpm run typecheck        # type-check without emitting
pnpm fix                  # format/lint via ultracite (biome)
pnpm check                # check for lint issues
```

E2E tests hit real provider APIs and require keys in `.env` or exported in shell. Unit tests are in `src/__tests__/*.test.ts`, e2e tests in `src/__tests__/e2e/*.e2e.test.ts`.

## Architecture

This is `@speech-sdk/core` — a universal TTS SDK (Node, Edge, Browser) with a single public function `generateSpeech()` and a provider abstraction for multi-provider support.

**Core flow:** `generateSpeech()` → `resolveModel()` → `provider.generate()` → `SpeechResult`

- `src/generate-speech.ts` — the public API entry point; handles retry logic via `p-retry`
- `src/resolve-provider.ts` — bare `"provider/model"` strings resolve to the gateway provider; `ResolvedModel` instances pass through unchanged
- `src/providers/gateway/index.ts` — `SpeechGatewayProvider` + `createSpeechGateway()`; proxies inline-mode requests to `api.speechbase.ai`. Aggregates every built-in provider's `models[]` under namespaced ids (`openai/tts-1`) so capability checks work through the gateway
- `src/speech-provider.ts` — `SpeechProvider` interface all providers implement
- `src/speech-result.ts` — `DefaultGeneratedAudioFile` with lazy base64 conversion
- `src/provider-utils.ts` — shared `resolveApiKey()` and `handleErrorResponse()`
- `src/providers/openai/` and `src/providers/elevenlabs/` — provider implementations

**Two paths to a provider** (chosen by how the caller passes `model`):
- String (`"openai/tts-1"`) → routes through `SpeechGatewayProvider`; needs `SPEECHBASE_API_KEY` (legacy `SPEECH_GATEWAY_API_KEY` still honored).
- Factory (`createOpenAI()("tts-1")`) → calls the provider directly; reads the per-provider env var (`OPENAI_API_KEY`) unless an explicit `apiKey` is passed to the factory.

**Gateway invariant:** when routing through `SpeechGatewayProvider`, the SDK is a thin REST wrapper. A call made via the SDK must be byte-equivalent to the same call made via `curl` against `api.speechbase.ai`. The SDK does not add behavior on the gateway path — no client-side recovery, no client-side enrichment, no synthesizing fields from caller input that weren't on the wire, no fallbacks. The gateway server owns its contract; the SDK is a transport. Any new feature must work identically whether the caller uses the SDK or hits the REST API directly.

**Adding a new provider:**
1. Create `src/providers/<name>/index.ts` with a `<Name>SpeechProvider` class implementing `SpeechProvider` and a `create<Name>()` factory.
2. Add subpath export in `package.json` under `exports`.
3. Register the provider in `aggregatedModels()` in `src/providers/gateway/index.ts` so its models are discoverable through the gateway path.
4. Implement `resolveOutputFormat(modelId, output)` so the SDK can request the user's chosen output format natively from the API. Return `{ providerOptions, expectedMediaType }` for each format the provider supports; for formats the provider can't produce natively, return options that yield a decodable wav/pcm so the SDK can convert via mediabunny. Return `undefined` for unknown model ids. The SDK never decodes compressed audio — providers must produce wav/pcm for any format the user requests that isn't natively available.
5. Implement `getStitchOptions(modelId)` so the conversation stitch path can request decodable wav/pcm regardless of user format preference (the stitch pipeline always operates on raw samples).

## Key Conventions

- ESM-only (`"type": "module"` in package.json); use `.js` extensions in imports
- TypeScript strict mode, target ES2022
- `providerOptions` are passed through to provider APIs untransformed
- Tests use vitest with globals enabled
- Run `pnpm fix` before committing to ensure formatting compliance

## Versioning & Releases

Follow semver. Prereleases use the canonical `0.N.M-alpha.K` form so they collapse cleanly into the corresponding stable `0.N.M`.

- **Stable**: `0.8.0`, `0.8.1`, `0.9.0`. Published to npm `latest`.
- **Prerelease**: `0.9.0-alpha.0`, `0.9.0-alpha.1`, … all pre-patches of the same target `0.9.0`. Published to npm `next`.
- When the next stable is cut, drop the suffix: `0.9.0-alpha.3` → `0.9.0`. Do **not** keep incrementing the minor/patch on the alpha track (e.g. `0.9.0-alpha`, `0.9.1-alpha`, `0.9.2-alpha`) — that creates phantom stable versions that never shipped and confuses npm's version ordering.
- Bump the alpha counter (`-alpha.K`), not the minor/patch, between prereleases of the same target.
- Breaking changes bump the minor while pre-1.0 (`0.8.0` → `0.9.0`); features alone can ride a patch on a stable line if no API changes.

## Code Standards

Formatting and linting enforced by Biome via ultracite. Husky pre-commit hook runs tests and lint automatically.

### TypeScript

- Prefer `unknown` over `any`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Jellypod-Inc/speech-sdk](https://github.com/Jellypod-Inc/speech-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
