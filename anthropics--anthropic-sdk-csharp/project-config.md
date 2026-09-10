---
trigger: always_on
description: Context for contributors (human or AI) on how this SDK is put together, plus the things reviews most often come back to. Setup instructions live in [CONTRIBUTING.md](CONTRIBUTING.md).
---

# Working in this repository

Context for contributors (human or AI) on how this SDK is put together, plus the things reviews most often come back to. Setup instructions live in [CONTRIBUTING.md](CONTRIBUTING.md).

## Overview

- The official C#/.NET SDK for the Claude API. Much of it is generated from the OpenAPI spec, with hand-written helpers, adapters, credentials, and platform clients layered on top.
- Generated files are safe to edit. New generator output is git-merged with custom changes rather than overwriting them, so fix things where they live; a collision is just a merge conflict.
- Bugs that originate upstream are worth flagging as such even when a local patch lands first: a real API response the SDK's generated types can't deserialize or validate (a missing "required" field, an unknown enum member, a nullability mismatch) points at the OpenAPI spec, and broken generated infrastructure (retries, SSE parsing, serialization, disposal) points at the generator — fixing it there fixes every SDK.
- PRs target `next`. `main` only moves when a release is cut. `next` moves quickly and its history is rewritten around releases, so rebase onto the current `next` before asking for review — a stale base shows up as dozens of unrelated "changed" files.
- Package versions (`VersionPrefix`, `.release-please-manifest.json`) and the per-package `src/*/CHANGELOG.md` files are written by release automation. What gets built, packed, and published is whatever is in the root `Anthropic.sln`; a new package additionally needs entries in `release-please-config.json` and the manifest, plus a line in the hand-maintained root `CHANGELOG.md` index.

## Build, test, lint

- The entry points are `./scripts/{bootstrap,build,test,format,lint}` (see CONTRIBUTING.md); `build` compiles `Anthropic.sln` and `examples/Anthropic.Examples.sln`. Run `format` and `lint` before pushing changes to C# or project files.
- Lint is `dotnet csharpier check .` (pinned version, via `dotnet tool restore`) plus `dotnet format style`/`analyzers` at info severity, so even IDE/CA suggestions fail it; CI, not a local run, is the source of truth. Prefer fixing a finding over suppressing it, and comment any suppression that's genuinely needed.
- `.cs` files use CRLF line endings (`.editorconfig`); csharpier flags LF files, which occasionally arrive via codegen, and `./scripts/format` fixes them.
- IDE churn (`.sln` header rewrites, added `this.` qualifiers, editor-specific `.gitignore` entries, whitespace-only diffs) is best reverted before committing.
- Warnings are errors and nullable reference types are on throughout `src/`. Other settings (target frameworks, `LangVersion`, `ImplicitUsings`) come from `src/Directory.Build.props` and several projects override them — `ImplicitUsings`, for instance, is off in `Anthropic` but on in most platform packages and the examples — so follow the project you're in rather than assuming.
- The libraries multi-target modern .NET plus `netstandard2.0`, so build with an SDK at least as new as the newest target framework. Tests run on .NET and on `net472`; the `net472` run is what exercises the `netstandard2.0` build (on Windows in CI), so both sides of a `#if` get tested somewhere.
- `./scripts/test` starts a Node-based mock API server first (`./scripts/mock`, port 4010, or point `TEST_API_BASE_URL` elsewhere). Only the generated `Services` tests and their `*MultiClientTest` wrappers talk to it; everything else, including the generated `Models` tests, runs without it. Live tests against real cloud providers only run with `ANTHROPIC_LIVE=1`.
- CI's `./scripts/detect-breaking-changes` compiles `main`'s generated `Services`/`Models` tests against your branch. It only sees the generated API surface, so a break in hand-written public API (helpers, adapters, credentials, platform clients) has to be called out by the author; and if the job fails the same way on plain `next`, it's pre-existing drift awaiting release rather than something the PR did.
- Example projects target `net8.0`, should read credentials from the environment, and need adding to `examples/Anthropic.Examples.sln`: `./scripts/build` compiles the examples through that solution, and `./scripts/lint` fails if a project under `examples/` is missing from it. New user-facing helpers usually come with an example; user documentation lives outside this repo (see CONTRIBUTING.md), so note in the PR what needs documenting.

## Multi-targeting

- `#if NET` / `#if !NET` is the convention for TFM splits, wrapped around just the fragment that differs. The `*_OR_GREATER` symbols are easy to misjudge, so check which targets define a symbol before relying on one.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anthropics/anthropic-sdk-csharp](https://github.com/anthropics/anthropic-sdk-csharp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
