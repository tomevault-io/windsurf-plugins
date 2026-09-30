---
trigger: always_on
description: Audience: agents editing this repository. Agents *calling* the REST API read `SkillsForUnity/unity-skills~/SKILL.md` instead — that file is the shipped protocol doc (8,192-byte hard cap; new content goes under `references/`) and the authoritative description of operating modes, permission endpoints, batch/diff semantics and observability. Read it before touching anything the client sees.
---

# UnitySkills — instructions for AI agents developing this repo

Audience: agents editing this repository. Agents *calling* the REST API read `SkillsForUnity/unity-skills~/SKILL.md` instead — that file is the shipped protocol doc (8,192-byte hard cap; new content goes under `references/`) and the authoritative description of operating modes, permission endpoints, batch/diff semantics and observability. Read it before touching anything the client sees.

| Field | Value |
|------|----|
| Version | 2.8.4 |
| Stack | C# Unity Editor plugin (UPM `com.besty.unity-skills`) + Python client |
| Unity | 2022.3+, verified on 6000.x |
| License | MIT |

## Architecture

`AI agent → unity_skills.py → HTTP localhost:8090-8100 → SkillsHttpServer (producer-consumer) → SkillRouter (reflects [UnitySkill]) → 56 *Skills.cs / 54 SkillCategory / 805 skills`. `WorkflowManager` = persistent undo/rollback, `RegistryService` = multi-instance discovery. 56 vs 54: `BatchSkills.cs` registers into Workflow + Validation, `DiagnoseSkills.cs` into Debug. The per-category table lives in `README.md`; `/skillcheck` keeps every count in sync — do not hand-edit counts.

- **Threading (hard rule)**: the HTTP thread only enqueues; the Unity main thread drains the queue from `EditorApplication.update`. Zero `UnityEngine.*`/`UnityEditor.*` calls off the main thread.
- **Every request, including GET `/health` `/jobs/{id}` `/skills`, goes through that main-thread queue** (≤20 per tick). A skill that blocks the main thread also freezes liveness; the Python client polls `GET /jobs/{id}` instead of `job_wait` for this reason.
- **No auth + wildcard CORS is intentional** (loopback bind is the accepted boundary). Do not report "add auth / tighten CORS / validate Origin" as a security finding.
- Optional-package modules (ProBuilder, XR, Netcode, YooAsset, DOTween, PrimeTween, Behavior, HybridCLR, Addressables, QFramework) detect their dependency and return `MISSING_PACKAGE`; URP-family modules (Volume/PostProcess/Decal/URP) compile to same-named `NoURP()` stubs without `com.unity.render-pipelines.universal`. QFramework has no UPM package, so detection is by reflected anchor type and it must not declare `RequiresPackages`.
- 28 advisory modules under `unity-skills~/skills/` are documentation only: no REST skills, no C# stub, ever.

Key files (`SkillsForUnity/Editor/`):
- `Skills/`: `SkillsHttpServer.cs`, `SkillRouter.cs`, `SkillPlanningService.cs` (/plan + dryRun engine, not a skill), `UnitySkillAttribute.cs`, `SkillErrorResponse.cs` + `SkillErrorCode.cs`, `SkillsLogger.cs` (single source of `Version`), `SkillsModeManager.cs`, `SkillsAuditLog.cs`, `ConfirmationTokenService.cs`, `WorkflowManager.cs`, `RegistryService.cs`, `GameObjectFinder.cs`, `BatchExecutor.cs`, `SkillInstaller.cs`, `AgentInstructionService.cs`, `UnityCliService.cs`, `ClientProcessResolver.cs` (caller identity via TCP port → PID → parent chain), `*Skills.cs` ×56.
- `Locales/{en,zh-CN,ru}.json`: all UI strings.
- `UI/`: `UnitySkillsWindow.{cs,uxml,uss}`, `Controllers/*.cs`, `Tabs/*.uxml`, `EditorUiScheduler.cs`, `ShortcutActions.cs`, `UISkillsFontIncrementalUpdater.cs`, `AuditLogWindow`, `AllowlistPickerWindow`, `UnityCliWindow`.
- `SkillsForUnity/unity-skills~/`: shipped template — `SKILL.md`, `scripts/unity_skills.py`, `skills/` (82 module docs: 54 REST + 28 advisory), `references/`.

## Rules

### Editor UI — UI Toolkit only
- No IMGUI (`OnGUI`, `EditorGUILayout`, `GUILayout`, `OnInspectorGUI`). New UI = `.cs + .uxml + .uss` in `Editor/UI/`, loaded via `Packages/com.besty.unity-skills/Editor/UI/...` path constants; in `CreateGUI()` add the USS then `CloneTree`; query nodes with `rootVisualElement.Q<T>("name")`.
- Periodic refresh only via `EditorUiScheduler.RepeatSafe(element, ms, body)`. Never bare `schedule.Execute().Every()` mutating the tree (issue #44: throws during repaint and loops) and never `EditorApplication.update` polling. Expensive data refreshes on tab activation / button / value change, not on the tick.
- One controller per UXML subtree: `XxxController(VisualElement root, EditorWindow owner)`; the window only assembles. New tab = `Tabs/X.uxml` (+ `.meta` with fresh GUID) + `Controllers/XTabController.cs` + one `MainTabDefinition` appended to `UnitySkillsWindow.MainTabs`. Copy `HistoryTabController`, the smallest complete example.
- `[MenuItem("Window/UnitySkills")]` is a leaf and must stay the only item under that prefix (Unity swallows a leaf that coexists with a submenu). Secondary panels open via in-panel buttons + shortcuts only.
- Shortcuts: a `[Shortcut]` static method in `Editor/UI/ShortcutActions.cs` plus a `Commands` entry; unbound by default; bindings live in ShortcutManager, not EditorPrefs.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Besty0728/Unity-Skills](https://github.com/Besty0728/Unity-Skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
