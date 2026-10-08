---
trigger: always_on
description: - Code task: read [context overview](Context~/HANDOFF.md); consult only relevant sections of [feature decisions](Context~/DECISIONS.md). These files supplement these rules; they are not a task queue.
---

# WhimTex — agent rules

## Start here

- Code task: read [context overview](Context~/HANDOFF.md); consult only relevant sections of [feature decisions](Context~/DECISIONS.md). These files supplement these rules; they are not a task queue.
- Browser AI / clipboard JSON / standalone HLSL: [AI_AUTHORING.md](AI_AUTHORING.md) → [contract](Documentation~/AI/README.md) → [examples](Documentation~/Examples/Clipboard/README.md).
- Edit an open compositor: read [whimtex-live skill](Skills~/whimtex-live/SKILL.md) first, then [LiveAgentAPI](Documentation~/LiveAgentAPI.md) as needed.
- Edit a compositor by path: [AgentAPI](Documentation~/AgentAPI.md).
- Texture/VFX authoring references: [internal agent samples](Samples~/AgentTextures/README.md). 38 procedural JSON recipes with individual PNG previews, at most 256 pixels on the longest axis; no bundled TIFF duplicates. Read the manifest descriptions/tags and only the relevant examples.
- Do not load all references for an unrelated task. These files describe rules and decisions, not Git status or a task queue.

## Identity and compatibility

- Product: WhimTex. Package: `com.dcfapixels.whimtex`. Namespace: `DCFApixels.WhimTex`; assemblies: `DCFApixels.WhimTex*`.
- Preserve `whimtex_*` commands, `whimtex-live` skill ID, `Temp/WhimTex/` preview path, `whimtex-` USS prefix.
- Preserve existing `MovedFrom` rename markers. Past permission to break a specific format is not permission for future breaking changes.
- Repository: `DCFApixels/WhimTex`; site: https://dcfapixels.github.io/WhimTex/.
- Never increase version without an explicit request, including when committing/pushing.
- Never mention or use names of external graphics editors in code, identifiers, comments, UI, tests/examples, documentation, changelogs, commit messages, pull requests or release notes. Describe behavior generically; do not use editor names as shorthand or comparisons.

## Cross-platform code

- Cross-platform code is the default. OS-specific system-library calls, platform-specific native binaries, explicit CPU/GPU instruction-set intrinsics and hardware/vendor-specific implementation paths may be researched and proposed, with their portability tradeoffs explained, but require explicit user approval before implementation or use. A portable fallback does not waive this requirement; a general optimization request is not approval for such an exception.
- Use portable C# and supported cross-platform Unity APIs. Burst/Jobs are allowed for portable algorithms; automatic compiler/backend targeting is not permission to write hardware-specific branches or intrinsics.
- These restrictions also apply to new test and benchmark code. Existing platform-specific diagnostic helpers are not a precedent or permission to expand their use.

## Execution and safety

- Do not introduce or extend solutions that access Unity internal/non-public APIs through reflection without explicit user approval. This includes reflected delegates, compatibility wrappers, and test or diagnostic code. Prefer supported public APIs; if no suitable alternative exists, explain the limitation and request approval before implementation. Existing reflection-based integrations and general feature requests are not authorization for new internal-API dependencies.
- Follow root project AGENTS.md. Compile only through the connected Unity Editor/Pipeline; no standalone MSBuild or dotnet build. One compilation at a time; check completion/errors. Player builds require a separate request.
- Explicitly target the intended Unity project. Missing tools do not authorize installing packages, starting another Editor or changing unrelated projects/assets.
- For compositor authoring use `WhimTexApi`, not generated YAML or simulated clicks. Source-code work does not require the authoring API.
- Prefer linked built-in/project FX: discover them with `whimtex_fx_catalog`, add through `presetId`, and adjust parameters with `set`. Do not copy unchanged preset HLSL into `code`; reserve inline code for custom algorithms or intentionally independent variants. User-library embedding remains the API's normal behavior.
- Inspect existing documents before path-based edits; use IDs and `@aliases`, not display names. Validate unfamiliar batches with `dryRun:true`; check inner API success, not just CLI transport success.
- After ambiguous timeouts/save failures, inspect before retrying. Adds and strokes are not idempotent.
- Scope assets to the requested output. Do not overwrite unrelated files, change source import settings or delete work to recover from a failed command.
- Verify image-authoring results with an actual rendered preview; no `-nographics`.

## Live generation

- `whimtex_assistant_begin` reserves a named layer and captures context before lengthy generation. Preserve the user's subsequent placement/name/visibility changes.
- Choose source sampling and strict/approximate selection from the request; do not impose one policy on every generation.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DCFApixels/WhimTex](https://github.com/DCFApixels/WhimTex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
