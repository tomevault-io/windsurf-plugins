---
trigger: always_on
description: - `Runtime/ComponentData/` contains camera/light companion data (`VividAdditionalCameraData`, `VividAdditionalLightData`) and should stay aligned with editor inspectors in `Editor/ComponentEditor/`.
---

# Repository Guidelines

## Project Structure & Module Organization
- `Runtime/ComponentData/` contains camera/light companion data (`VividAdditionalCameraData`, `VividAdditionalLightData`) and should stay aligned with editor inspectors in `Editor/ComponentEditor/`.
- `Runtime/Extension/CoreRP/` contains Core RP extension glue plus the `VividRP.CoreRP.Runtime.asmref`; keep assembly references intact when moving these files.
- `Runtime/RenderPipeline/` contains the SRP entry points and settings objects (`VividRenderPipeline`, `VividRenderPipelineAsset`, `VividRenderPipelineGlobalSettings`).
- `Runtime/RenderGraph/` contains the reflection-driven pass model, pass recorder, frame context types, preview/history registries, and resource descriptors used at runtime.
- `Runtime/RenderPass/` contains concrete passes, currently grouped under `Core/` and `Example/`; new passes typically derive from `RasterPass`, `UnsafePass`, or `ComputePass`.
- `Runtime/Utility/PipelineResource/` plus `Runtime/Resources/PipelineResources.asset` implement package resource lookup based on `[PipelineResource]` and `[ResourcePath]` attributes.
- `Editor/RenderGraph/` contains the GraphToolkit-based RenderGraph editor, validators, importers, pass-compilation utilities, node data types, drawers, navigation helpers, and pass-node registry generation.
- `Editor/RenderGraph/GeneratedRenderPassNodes.g.cs` is generated code. Update the generator, registry builder, or runtime pass types instead of editing this file by hand.
- RenderGraph authoring assets use the `.vrdg` extension and are imported into `RenderGraphData` assets by `Editor/RenderGraph/RenderGraphImporter.cs`; change importer/compiler behavior carefully and keep import-time tests current.
- `Editor/PipelineResource/` and `Editor/RenderPipeline/` contain editor automation such as resource syncing and global settings hooks.
- `Editor/ComponentEditor/`, `Editor/Material/`, `Editor/Shader/`, and `Editor/VolumeEditor/` contain custom inspectors, shader GUIs, and editor-only shader assets; keep runtime/editor boundaries clean.
- `Shaders/` is a top-level package folder with package shaders and the `VividRP.Shaders` assembly; shader assets are not stored under `Runtime/Shaders/`.
- `Documentation/` contains the current package notes for RenderGraph editor usage, resource descriptors, and acceleration-structure support; keep higher-level workflow docs there.
- `Tests/Editor/` contains the current committed suite through the `VividRP.Editor.Tests` assembly, including pass, node, drawer, importer, history, component, and editor coverage. Add `Tests/Runtime/` only when runtime-specific coverage is needed.
- Do not manually create or edit `*.meta` files; let Unity generate and maintain them automatically.
- Unity `.meta` files, generated assets, and package-relative paths must stay in sync when moving or renaming files. If the package path or package name changes, update both `Editor/PipelineResource/PipelineResourceUpdater.cs` and `Editor/RenderGraph/RenderPassNodeRegistryGenerator.cs`.
- The repository currently uses both `Packages/com.af8a2a.vividrp/...` and `Packages/VividRP/...` path constants; do not “fix” only one side during refactors—audit all package-relative paths together.

## Build, Test, and Development Commands
- Open the package through the Unity project root `E:\VividRP_Reborn` using Unity `6000.5.0a7` or a compatible `6000.5` build.
- Run the current EditMode suite with Unity Test Framework:
  `Unity.exe -batchmode -projectPath "E:\VividRP_Reborn" -runTests -testPlatform EditMode -testResults Logs/editmode-results.xml -quit -logFile Logs/editmode.log`
- There are no committed PlayMode tests yet. Add the relevant test assembly before documenting or relying on a PlayMode batch command.
- Quick pass/resource search: `rg "IRenderPass|RenderGraphResource|PipelineResource|ResourcePath" Runtime Editor Tests`
- Quick editor/codegen search: `rg "GeneratedRenderPassNodes|BuildRegistrations|RegisteredPassTypeName" Editor Runtime Tests`
- Quick package path audit: `rg "Packages/VividRP|Packages/com.af8a2a.vividrp|com.af8a2a.vividrp" Runtime Editor Tests package.json`

## Coding Style & Naming Conventions
- Use 4-space indentation, braces on new lines, and small focused methods.
- Match namespaces to area, for example `VividRP.Runtime`, `VividRP.Runtime.RenderPass.Core`, `VividRP.Editor.RenderGraph`, and `VividRP.Editor.Tests`.
- Preserve reflection-driven contracts: runtime pass resource fields are discovered via `[RenderGraphResource]`, and editor port generation plus preview lookup depend on those field names, access flags, and field types.
- Preserve serialized field names in authoring/runtime data models unless you also add an explicit migration path. `RenderGraphData`, `RenderGraphPassDefinition`, and `PipelineResourcesContainer` are serialized assets that survive importer/editor updates.
- Put `[RenderGraphResource]` on fields, not properties. The collector reflects instance fields across the inheritance chain and ignores null resource values.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [af8a2a/VividRP](https://github.com/af8a2a/VividRP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
