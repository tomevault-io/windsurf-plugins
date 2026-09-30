---
trigger: always_on
description: This file applies to the entire plugin repository. Read it before changing code, build rules, bundled libraries, or documentation. Follow explicit user scope and any more specific directory instructions. Keep this guide current when architecture or development procedures change.
---

# Agent guide: UE5_Spout2_DX12

This file applies to the entire plugin repository. Read it before changing code, build rules, bundled libraries, or documentation. Follow explicit user scope and any more specific directory instructions. Keep this guide current when architecture or development procedures change.

## Purpose and supported scope

This is a Windows/Win64 Unreal Engine runtime plugin for GPU texture sharing through Spout and D3D11On12 while Unreal uses the D3D12 RHI. It provides Blueprint-spawnable sender and receiver actor components. It is a plugin repository, not a standalone Unreal project.

The inspected baseline is `Spout2_DX12.uplugin` version name `2.1.1` (numeric `Version: 3`), with one `Runtime` module, `Spout2_DX12`, loaded at `Default`. The descriptor permits content, but no Content directory or sample `.uproject` is currently tracked.

`README.md` reports Windows/DX12 testing on UE 5.2.1, 5.4.4, 5.6.1, 5.7.2, 5.8 Preview, and 5.8. These are historical project claims, not proof that a new change works on those versions. Do not infer DX11, Vulkan, Linux, macOS, headless server, or cross-adapter support from conditional compilation or the presence of other SDK libraries.

## Start each task

1. Read the request and inspect `git status --short` and relevant diffs. Preserve existing work; do not reset, clean, or overwrite unrelated changes.
2. Read this guide, `README.md`, the plugin descriptor, and the files relevant to the requested behavior. Source and build rules take precedence when prose disagrees with implementation.
3. Read `local plans.md` if present. It is a private working notebook, not a committed specification or authorization to implement every backlog item. Create it if absent when planning substantial work.
4. Establish the Unreal version, host project, active RHI, execution mode, and sender/receiver configuration relevant to the task. Do useful inspection before asking for information that cannot be inferred locally.
5. For nontrivial work, record scope, acceptance criteria, steps, validation, and unresolved questions in the local plan. Keep changes limited to the request.

Use `rg` / `rg --files` for navigation. Useful entry points:

```powershell
git status --short
rg --files Source Config
rg -n 'ENGINE_MINOR_VERSION|WITH_EDITOR|PLATFORM_WINDOWS' Source/Spout2_DX12
rg -n 'StartBroadcast|StartReceiving|StopBroadcastInternal|StopReceivingInternal' Source/Spout2_DX12
rg -n 'Fence|Flush|AcquireWrappedResources|ReleaseWrappedResources' Source/Spout2_DX12/Private
```

If Git reports dubious ownership for this known checkout, use a command-scoped exception, for example `git -c safe.directory=D:/UE5_Spout2_DX12 status --short`. Use the actual checkout path. Do not change global Git trust settings or trust every directory to work around it.

## Repository map

| Path | Responsibility |
| --- | --- |
| `Spout2_DX12.uplugin` | Plugin identity, release metadata, and runtime module registration. |
| `Source/Spout2_DX12/Spout2_DX12.Build.cs` | Active Unreal dependencies, SDK include/lib paths, delay loading, and packaged DLL staging. |
| `Source/Spout2_DX12/Public/Spout2_DX12.h` | Module interface and `LogSpoutSender` / `LogSpoutRX` declarations. |
| `Source/Spout2_DX12/Private/Spout2_DX12.cpp` | Module startup/shutdown and explicit `SpoutDX12.dll` loading. |
| `Source/Spout2_DX12/Public/SpoutSenderComponent.h` | Sender Blueprint API, configuration, staging slots, fences, and viewport callback state. |
| `Source/Spout2_DX12/Private/SpoutSenderComponent.cpp` | Sender lifecycle, source resolution, editor ownership, GPU copies, and Spout publication. |
| `Source/Spout2_DX12/Public/SpoutReceiverComponent.h` | Receiver Blueprint API, render targets, connection state, statistics, and fence state. |
| `Source/Spout2_DX12/Private/SpoutReceiverComponent.cpp` | Sender discovery, shared-resource reception, D3D11On12 copies, output publication, and cleanup. |
| `Source/Spout2_DX12/Public/SpoutSenderSource.h` | Serialized `ESpoutSenderSourceType` enum: RenderTarget, GameViewport, EditorViewport. |
| `Source/Spout2_DX12/Public/SpoutWorldPolicy.h` | Shared `ESpoutWorldBootstrapPolicy` enum. Each component implements its own policy checks. |
| `Source/Spout2_DX12/Public/Spout2BlueprintLibrary.h` and matching private `.cpp` | Empty Blueprint function-library scaffold; component methods currently implement the useful API. |
| `Source/ThirdParty/include/` | SDK headers used by the active runtime module. |
| `Source/ThirdParty/lib/Win64/` | SDK libraries; the active module links `Spout.lib` and `SpoutDX12.lib`. |
| `Source/ThirdParty/bin/Win64/` | SDK DLL sources; the active module stages `Spout.dll` and `SpoutDX12.dll`. |
| `Source/ThirdParty/Spout2_DX12Library/` | Alternate SDK layout and external-module Build.cs. It is not a dependency of the active runtime Build.cs. Do not assume changes here affect the plugin. |
| `Binaries/Win64/` | Tracked Spout DLLs plus prebuilt UnrealEditor plugin DLL, PDB, and `.modules` metadata. These are engine/build-specific artifacts. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GPUbrainStorm/UE5_Spout2_DX12](https://github.com/GPUbrainStorm/UE5_Spout2_DX12) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
