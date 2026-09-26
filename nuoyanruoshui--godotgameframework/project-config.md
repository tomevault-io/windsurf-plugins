---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**GGF** (Godot Game Framework) — **Godot 4.7 + C# (.NET 8)** port of [Game Framework](https://gameframework.cn/) (Jiang Yin). Modular architecture: Event, FSM, Procedure, Resource, Entity, UI, Audio, Localization, ObjectPool, DataTable, DataNode, Setting, WebRequest, Download, Debugger, Archive.

> 📚 **Per-system deep-dive docs live in `docs/`** (FrameworkCore / Event / Fsm / Procedure / Debugger / Resource / Entity / ObjectPool / UI / Sound / Scene / DataTable / DataNode / Setting / Localization / WebRequest / Download / Archive + 资源热更审计; C# 程序集热更方案已搁置等待华佗团队适配). See `docs/README.md` for the index. Prefer those docs over this file for system details.

- **Godot .NET SDK**: `Godot.NET.Sdk/4.7.0` (NuGet)
- **Build**: `cd GodotProject && dotnet build`
- **Add .cs files**: `"<godot_exe>" --build-solutions --path GodotProject --no-window -q`
- **Open editor**: `"<godot_exe>" --path GodotProject --editor`
- **Godot path**: `D:\Godot\Godot_v4.7-stable_mono_win64\Godot_v4.7-stable_mono_win64\Godot_v4.7-stable_mono_win64.exe` (note: the exe is nested two directory levels deep — `Godot_v4.7-stable_mono_win64/Godot_v4.7-stable_mono_win64/`). Bash on Windows here needs forward slashes.
- **Active game project**: `TheGame/`
- **No test framework detected** — game is runtime-only (no test files found)
- **Rendering**: D3D12 (Forward Plus), **Physics**: Jolt Physics (3D), **Stretch**: canvas_items / expand

## Dual-Layer Architecture

The framework has a strict **two-layer separation** mirroring the original Game Framework design. **Key rule:** `GameFramework/` knows nothing about Godot. `GodotGameFrameworkCore/` depends on both `GameFramework/` and Godot — new systems put interface/logic in `GameFramework/` and Godot bridge in `GodotGameFrameworkCore/`.

```
GodotProject/
  Framework/
    GameFramework/                  ← Pure C# modules (zero Godot dependency)
      Base/                         ← GameFrameworkEntry, GameFrameworkModule, ReferencePool, EventPool
      Fsm/                          ← State machine system
      Procedure/                    ← Procedure (game state) manager
      Entity/ UI/ Sound/ Scene/     ← Manager interfaces + logic (no Godot types)
      DataNode/ ObjectPool/
      Resource/                     ← IResourceManager + 加载回调委托/状态枚举/Constant
      Debugger/ Download/           ← Debugger windows, download manager
      Event/ Localization/          ← Event manager, localization system
      WebRequest/                   ← IWebRequestManager + WebRequestManager (TaskPool 调度、serialId、超时)
      Properties/ Utility/          ← Assembly info, text/compression utilities
    GodotGameFrameworkCore/         ← Godot runtime components
      Base/                         ← GF.cs facade, GameEntry, GameFrameworkComponent, GodotComponent
      Entity/ UI/ Sound/ Scene/    ← Godot bridge components (each delegates to the corresponding Manager)
      Resource/                     ← ResourceComponent, ResourceManager, PackVersionList, load tasks, IResourceLoadHelper/DefaultResourceLoadHelper
      Download/ WebRequest/         ← DownloadComponent (queue+resume+verify), WebRequestComponent (wraps IWebRequestManager + N HttpRequest agent helpers)
      HotUpdate/                    ← HotUpdateSafetyGuard (crash-safe hot update)
      DataNode/ Setting/ Localization/
      Event/ Fsm/ Procedure/ ObjectPool/          ← ObjectPool 含 ReferencePoolComponent（引用池严格检查策略）
      Debugger/                     ← UGF 风格运行时调试器（FPS 图标 + Console/Information/Profiler/Other 页签）
      Archive/                      ← ArchiveSystem<T,U> 通用存档系统（Catalogue + Data 分离）+ Rijindael（AES-256 加密，由 ArchiveSetting 配置）
      Config/ Variable/             ← GameFolderConstant, VarInt32/VarString/VarBoolean/VarSingle
      Json/                         ← Newtonsoft.Json helper (local .dll reference) + EasySave
      Lib/LubanLib/                 ← Luban runtime (ByteBuf, BeanBase, StringUtil)
      SingletonSystem/              ← SingletonNode<T> pattern
      Templet/                      ← Script generation templates (UIForm/Entity, Ge/Logic)
      Utility/                      ← PhysicsCheck2D, NodeExtension, DefaultLogHelper, GTween, LayerMask
  TheGame/                          ← Active game project
    MainPack/                       ← Main package (shared core infrastructure)
      Scripts/
        ObjectPool/                 ← NodePool, NodeObject, PoolContainer
        Procedure/                  ← ProcedureLaunch, ProcedureUpdate, ProcedurePrelode, ProcedureGame
        Resources/                  ← EntityGroup, SoundGroup, UIGroup, NodePoolConfig, ScriptGenerateRes, UpdateSettingRes
        UI/                         ← LoadingForm, QuestionTips (shared UI)
      Fonts/                        ← simhei.ttf
      Resources/                    ← .tres config resources (EntityGroupRes, UIGroupRes, etc.)
      Themes/                       ← MainThemes.tres
      UI/                           ← .tscn scene files (LoadingForm, QuestionTips)
    GameScripts/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nuoyanruoshui/GodotGameFramework](https://github.com/nuoyanruoshui/GodotGameFramework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
