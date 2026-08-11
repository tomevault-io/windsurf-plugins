---
trigger: always_on
description: - Run `npm run check`.
---

# Repository rules

## Before handing off changes

- Run `npm run check`.
- Run the narrowest relevant benchmark or fixture verification when changing benchmark code.
- Keep every implementation and test dependency in `devDependencies`. The published package must have
  no runtime dependency tree.
- Production codecs and processing code must be implemented in this repository. Do not bundle,
  vendor, copy, or runtime-import a third-party implementation to disguise a dependency as local
  package contents. Dev dependencies may be used only as test or benchmark oracles.

## Releases

- When preparing, verifying, publishing, or auditing a release, follow the `release-purejsimage`
  repository skill in `.agents/skills/release-purejsimage/SKILL.md`.
- Only release manager Aaron Decker (`a-r-d`) publishes releases. Do not infer permission to change
  versions, create tags, push release commits, publish to npm, or create GitHub releases.
- Never request, expose, or store release credentials or npm one-time passwords.

## Codec capability rollouts

- When adding, expanding, restricting, deprecating, or documenting a codec capability, follow the
  `rollout-codec-capability` repository skill in
  `.agents/skills/rollout-codec-capability/SKILL.md`.
- Treat `capabilities/manifest.json` as the only manually edited source for published codec support.
  Regenerate README, codec pages, website tables, public JSON, and compatibility expectations; do
  not edit generated capability surfaces directly.

## Experimental HEIF / HEIC

- HEIF/HEIC decoding must remain experimental and explicit opt-in because HEIC commonly carries
  HEVC/H.265 content that may be subject to third-party patent rights.
- Never export the HEIF/HEIC codec from the root package, include it in `allCodecs`, register it in
  the default browser demo, or activate it automatically on file detection. Keep the public codec
  available only through `purejsimage/codecs/experimental/heic`.
- The package may ship the first-party implementation under MIT, but project documentation must
  state that MIT grants no third-party patent rights and that users and distributors—including
  commercial products and services—must evaluate their own licensing obligations.

## Code style

- Write all source, benchmark, script, and test code in TypeScript with strict mode enabled.
- Never use `any`. Prefer narrow, clearly defined types, literal types, and discriminated unions over
  broad object shapes.
- Treat external input as `unknown` and narrow it with runtime checks. Do not bypass type safety with
  unchecked assertions or suppression comments.
- Prefer the smallest direct implementation that clearly solves the current problem.
- Use a few straightforward lines instead of introducing speculative layers, factories, or generic
  abstractions.
- Add an abstraction when it removes real repetition or enforces a real invariant, not because it may
  become useful later.
- Keep functions focused and data flow obvious. Avoid clever compression that makes code harder to
  review.
- In image-processing hot paths, minimize allocations, buffer copies, full-image materialization, and
  repeated pixel passes.
- Do not add features solely for API breadth. Optimize the workflows in the project specification and
  benchmark suite.

## Runtime portability

- PureJsImage must support both Node.js and modern browsers. Treat browser support as a release
  requirement for the shared codec, pipeline, transform, source, sink, and public API behavior—not
  as a best-effort compatibility layer.
- Keep the portable module graph free of Node built-ins, Node-only globals, and Node-only public
  types. Do not use `node:*`, `Buffer`, filesystem paths, or Node streams in modules reachable from
  `purejsimage/browser` or the codec entry points.
- Put Node path, Buffer, zlib, and temporary-file behavior behind the Node platform adapters. Put
  browser File/Blob, Uint8Array/Blob output, CompressionStream, and origin-private storage behavior
  behind the browser adapters. Do not duplicate codec or pixel-processing implementations between
  runtimes.
- Browsers cannot open arbitrary local path strings. Browser inputs should use File/Blob,
  ArrayBuffer, Uint8Array, fetched bytes, or an explicit `ImageSource`; browser outputs should use
  Uint8Array, Blob, or an explicit `ImageSink`.
- Resolve runtime capabilities and select adapters before entering codec or pixel hot loops. Never
  add repeated environment detection, polymorphic runtime branching, or platform lookups inside
  per-pixel, per-row, per-coefficient, or entropy loops.
- Feature-detect browser primitives such as CompressionStream and OPFS. Any fallback must be
  bounded, documented, and tested; when a safe bounded fallback is unavailable, throw an explicit
  `ImageError` instead of silently allocating a source-sized bitmap or loading a polyfill.
- Preserve existing Node performance and memory behavior when adding browser support. Browser
  portability work must not replace a bounded Node path with a generic full-frame implementation.
- For changes to public APIs, codecs, transforms, compression, sources, sinks, packaging, or runtime
  adapters, run `npm run browser:check` and add focused browser coverage. Changes to browser runtime

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [a-r-d/PureJsImage](https://github.com/a-r-d/PureJsImage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
