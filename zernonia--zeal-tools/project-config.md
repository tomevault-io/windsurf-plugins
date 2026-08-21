---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
pnpm install           # runs `nuxt prepare` via postinstall — required before test/build
pnpm dev               # Nuxt dev server on :3000
pnpm test              # vitest run (unit)
pnpm build             # nuxt build → .output (cloudflare_module preset)
pnpm lint              # eslint . (@antfu/eslint-config)
pnpm lint:fix          # autofix
pnpm preview           # wrangler dev against the built output
```

Single test file / single test name:

```bash
pnpm vitest run tools/qr-code-generator/core/encoder.test.ts
pnpm vitest run -t "boosts EC level"
```

CI (`.github/workflows/ci.yml`) runs **`pnpm test` and `pnpm build` only** — lint is not gated, so run it yourself before finishing. Deploys run through the **Cloudflare Workers Builds** git integration (worker `zeal-tools`); there is no Actions deploy workflow and no wrangler credentials are needed.

The background remover vendors **nothing**. Its weights stream from Hugging
Face on first use, and the ONNX Runtime wasm is emitted into `_nuxt/` as a
content-hashed asset by the bundler — leave `ort.env.wasm.wasmPaths` unset and
that happens automatically, version-locked to the installed `onnxruntime-web`.
Do not reintroduce a copy step: an earlier one drifted from the package and
404'd on the loader glue. Nothing is fetched on page load; verify with
`grep -c ort-wasm .output/public/tools/background-remover/index.html` (expect 0).

Two constraints shape the model choices. A Cloudflare static asset cannot
exceed **25 MiB**, and — the one that actually bites — **GitHub release assets
send no CORS headers at all**, so rembg's models cannot be fetched from a
browser however convenient they look. Hugging Face sends
`access-control-allow-origin: *`.

Three things about that tool are measured, not guessed, and must stay that way.
**Preprocessing and activation are per-model** (BiRefNet divides by a flat 255
and emits logits from about -15 to +8 needing a sigmoid; U²-Net divides by the
image's brightest channel and emits non-negative saliency needing a min-max
stretch) — using the wrong one yields a plausible, wrong matte, and both
branches stay unit-tested even though one model ships. **fp16 weights are not
faster** on the wasm backend, which has no fp16 acceleration, so they only
halve the download. And **WASM is not asynchronous**: it runs on the calling
thread, so inference on the main thread froze the page for 5.5s — hence
`ort.env.wasm.proxy = true`. That has a trap of its own: proxy mode *transfers*
input tensors, so any tensor reused across runs (the SAM embeddings) must be
copied per run or the second call throws `ArrayBuffer is already detached`.
Avoid ISNet despite its ideal 42 MiB int8 build: it is **AGPL-3.0**.
`onnxruntime-web` is the sole entry on CONTRIBUTING's runtime dependency
allow-list; verify it stays in one lazily imported chunk with
`grep -rl onnxruntime .output/public/_nuxt/`.

`package.json#packageManager` pins pnpm — without it Cloudflare Workers Builds picked a pnpm 8/9 that rejects a settings-only `pnpm-workspace.yaml` ("packages field missing or empty"). The `packages: [.]` entry there is belt-and-braces for the same reason; it does not make this a monorepo and does not change the lockfile.

pnpm enforces a supply-chain policy (`pnpm-workspace.yaml`). `semver@6.3.1` is exempted via `trustPolicyExclude` because the 6.x line predates npm trusted publishing; it arrives transitively through `nuxt → @vitejs/plugin-vue-jsx → @babel/core`. Don't widen that list without the same kind of justification.

## Git conventions

Commit messages are **Conventional Commits** — `type(scope): subject`, lowercase, imperative, no trailing full stop. Scope is the tool slug when the change belongs to one slice (`feat(stage-timer):`, `fix(background-remover):`), otherwise the area (`brand`, `seo`, `nav`, `home`). Split unrelated work into separate commits rather than one sweep; group files so each commit is a single coherent change.

Types in use: `feat`, `fix`, `refactor`, `perf`, `docs`, `test`, `chore`, `build`.

## Product invariants (the Zeal Promise — never violate)

- No sign-up, ever. No watermarks. No ads near download/copy actions.
- Privacy-first: process client-side wherever possible; tool inputs never reach our servers.
- The download/copy area is sacred — never block or crowd it.
- No dynamic/tracking QR codes (the accounts-and-paywall trap). No anonymous file hosting: with no sign-up it's an abuse magnet.
- "Request a tool" links point at `https://github.com/zernonia/zeal-tools/issues/new?template=tool-request.md`.

## Architecture

**Every tool is a self-contained Nuxt layer** under `tools/<slug>/`, composed in `nuxt.config.ts` via `extends`. `shared/` is also a layer. Read `CONTRIBUTING.md` — it owns the slice template, the zero-dependency policy, and the per-tool checklist. Follow it rather than restating it.

The two invariants everything hangs off:

1. **`core/` is pure and isomorphic** — no Vue, no DOM, no npm runtime deps. This is what lets one implementation serve the UI, the REST route, and MCP.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zernonia/zeal-tools](https://github.com/zernonia/zeal-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
