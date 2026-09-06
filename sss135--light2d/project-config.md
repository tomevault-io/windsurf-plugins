---
trigger: always_on
description: - This repository root **is** the Unity `6000.5.4f1` development/test project. The UPM package `com.sss13594.light2d` is embedded at `Packages/com.sss13594.light2d/` and is auto-discovered by Unity from there.
---

# Light2D Agent Guide

## Repository Shape

- This repository root **is** the Unity `6000.5.4f1` development/test project. The UPM package `com.sss13594.light2d` is embedded at `Packages/com.sss13594.light2d/` and is auto-discovered by Unity from there.
- The repo-root project's generated directories (`Library/`, `Temp/`, `Obj/`, `Logs/`, `Builds/`, `UserSettings/`, `TestResults/`) are git-ignored. Consumer-facing package payload excludes `Documentation~/` and `Samples~/` staging by trailing `~` and `.npmignore`.
- Open `X:\src\unity\Light2D` (the repo root) with Unity `6000.5.4f1` at `C:\Program Files\Unity\Hub\Editor\6000.5.4f1\Editor\Unity.exe`. Do not open a subfolder.
- `Packages/manifest.json` does NOT reference embedded packages via `file:` paths. `manifest.json` keeps `"testables": ["com.sss13594.light2d", "com.sss135.unity-mcp-streamlined"]` so both embedded packages expose their test assemblies.
- `Packages/com.sss13594.light2d/Documentation~/` holds the buyer-facing reference guides (`ARCHITECTURE.md`, `INSTALLATION.md`, `MIGRATION.md`, `BENCHMARKING.md` — runtime diagnostics only, `TROUBLESHOOTING.md`, and `FAQ.md`). Maintainer-only material lives at the repo root under `docs/`: `docs/TESTING.md` is the authoritative long-form of the test and host protocol summarized below, and `docs/BENCHMARKING.md` holds the benchmark fixtures and capture protocol. Keep host paths, editor paths, and MCP details out of the package — the package ships to Asset Store buyers.
- Use subagents for independent investigation or implementation work when available. Other agents and users may edit this checkout concurrently: re-read files immediately before editing, preserve unrelated changes, and never revert or overwrite work you did not author.

## Editing Rules

- Use `apply_patch` for manual file edits. Formatting or generated bulk output may use its owning tool.
- Preserve Unity `.meta` files and GUIDs. Never regenerate, delete, rename, or duplicate metas casually; serialized scenes, prefabs, materials, and the 2.0 migration depend on them.
- Do not edit generated solution/project files.
- Do not commit, amend, push, or create a pull request unless the user explicitly requests it.
- Keep package dependencies and the package version unchanged unless the task explicitly owns them.
- The legacy shader names, pass tags, properties, keywords, and serialized C# field names are compatibility contracts. In particular, preserve `_MainTex`, `_NormalTex`, `_ObstacleTex`, `_LightSourcesTex`, `_AmbientLightTex`, `_GameTex`, `_ObstacleMul`, `_EmissionColorMul`, `LightObstacle=True`, and the misspelled shader identifier `Light2D/Obstacle Texture Post Porcessor` unless the change includes a tested migration.
- SubShader ordering is load-bearing. In every dual-pipeline shader that Built-in composites through `Graphics.Blit` (the fullscreen passes: ambient, blur, overlay, and the obstacle post-processor), the Built-in `CGPROGRAM` SubShader MUST come first and the URP (`UniversalPipeline`) SubShader second. `Graphics.Blit` selects the first SubShader and ignores the `RenderPipeline` tag, so a URP-first ordering silently breaks Built-in ambient/blur/overlay composition — the defect that broke the whole Unity 6 port until it was fixed.
- The installed sample under `Assets/Samples/Light2D/Examples/` mirrors `Packages/com.sss13594.light2d/Samples~/Examples/` and must stay byte-identical; any script change to one must be mirrored to the other. `MapGenerator` builds one submesh and material/texture binding per source texture because Unity 6 has no legacy Sprite Packer atlas to merge block sprites into — do not reintroduce single-atlas assumptions.

## Architecture Boundaries

- Built-in rendering runs through `LightingSystem.OnRenderImage` and `LightingRenderResources`. Registered role components are drawn into the light-input `RenderTexture`s via `CommandBuffer.DrawMesh` (register-and-draw); there is no helper camera.
- URP rendering runs through `Light2DUniversalRendererFeature` and URP 17 RenderGraph. It derives culling and matrices from the source camera. `UniversalRendererData` and `Renderer2DData` support is implemented and behaviorally verified in batchmode through explicit single-camera render requests (the older ShaderGraph import defect is resolved on the 6000.5.4f1 host).
- Do not make the Built-in backend depend on RenderGraph or route URP rendering through `OnRenderImage`.
- Keep per-camera resources isolated. URP contexts and ambient histories are keyed by `LightingSystem`; serialized materials are cloned before runtime mutation.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SSS135/Light2D](https://github.com/SSS135/Light2D) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
