---
trigger: always_on
description: When adding guidance to this file, prefer discovery-first approaches (runtime queries, dynamic lookups, search patterns) over hardcoded values, paths, or version-specific details that can go stale.
---

When adding guidance to this file, prefer discovery-first approaches (runtime queries, dynamic lookups, search patterns) over hardcoded values, paths, or version-specific details that can go stale.

This is a Unity Project. We work inside the Assets folder (working directory).
Focused on Avatar Creation for VRChat.

**Primary approach**: Use the MCP bridge (`execute_csharp`) for all scene interaction — reading hierarchy, adding components, configuring settings, creating assets.
**File reads**: Freely read any file to explore assets, code, configs, YAML structure.
**File writes**: Fallback for creating/editing `.anim`, `.controller`, `.asset`, `.mat` files when the C# API is awkward or for bulk operations. When editing YAML directly, read an existing file of the same type first to understand the structure.
Do not create `.meta` files — Unity generates these automatically.

**Subagent limitation**: Subagents (Explore, Plan, etc.) do NOT have access to MCP tools like `execute_csharp`. Never delegate scene exploration or Unity Editor interaction to subagents — they will try to parse `.unity` files directly, which is unreliable. Always run MCP calls in the main conversation.

**Plan mode**: MCP tools (`execute_csharp`) are available during plan mode since the main agent retains all tools. Use MCP for scene exploration during planning — don't rely on parsing `.unity` YAML files.

The VRChat SDK are located at:
- Packages/com.vrchat.base
- Packages/com.vrchat.avatars

## MCP Bridge (`execute_csharp`)

C# snippets run inside the Unity Editor via the MCP `execute_csharp` tool. Key details:

- Runs on Unity main thread with a **30-second timeout**
- Must end with `return "some string";`
- Available usings: `System`, `System.Collections.Generic`, `System.IO`, `System.Linq`, `System.Text`, `UnityEditor`, `UnityEngine`, `UnityEngine.SceneManagement`
- All loaded assemblies are referenced (Unity, VRC SDK, VRCFury, project scripts)
- Each snippet is independent — chain multiple MCP calls for multi-step operations
- Always call `EditorUtility.SetDirty(obj)` on modified objects
- Save with `AssetDatabase.SaveAssets()` or `EditorSceneManager.SaveOpenScenes()`

## Workflow — Scene Exploration First

**Default**: work on the currently open scene unless the user specifies otherwise.

Start by exploring the scene hierarchy — find root GameObjects, locate the avatar, understand the structure before making changes.

- **Find avatar root** by enumerating all `VRCAvatarDescriptor` objects and recording each descriptor's full hierarchy path (`Root/Child/Avatar`), then target the avatar by exact path
- **Walk hierarchy** recursively from the avatar transform
- **Navigate bones** via `transform.Find("Armature/Hips/Spine/Chest/...")`
- **List components, blendshapes, materials** via standard Unity APIs (`GetComponents`, `sharedMesh.GetBlendShapeName`, `sharedMaterials`)

## Animation Clip Binding Reference

- **Blendshapes:** `EditorCurveBinding.FloatCurve("MeshPath", typeof(SkinnedMeshRenderer), "blendShape.ShapeName")`
- **GameObject active:** `EditorCurveBinding.FloatCurve("ObjectPath", typeof(GameObject), "m_IsActive")`
- Always `EditorUtility.SetDirty()` on modified objects; `Undo.RecordObject()` before changes

## VRC SDK Notes

The VRC SDK ships as compiled DLLs — no readable C# source for components.
- Whitelist source (readable C#): `AvatarValidation.cs` in both base and avatars packages
- PhysBone reference scene: `Packages/com.vrchat.avatars/Samples/Dynamics/Robot Avatar/Avatar Dynamics Robot Avatar PC.unity`

**Key C# namespaces:**
- `VRC.SDK3.Avatars.Components` — VRCAvatarDescriptor, VRCStation
- `VRC.SDK3.Dynamics.PhysBone.Components` — VRCPhysBone, VRCPhysBoneCollider
- `VRC.SDK3.Dynamics.Contact.Components` — VRCContactSender, VRCContactReceiver
- `VRC.SDK3.Dynamics.Constraint.Components` — VRCPositionConstraint, VRCRotationConstraint, etc.
- `VRC.SDK3.Avatars.ScriptableObjects` — VRCExpressionParameters, VRCExpressionsMenu

## Avatar Logic

### Write Defaults Convention

**Use Write Defaults ON** on all states.
- WD OFF in unmasked controllers causes FX to claim ownership of transforms/muscles from higher layers (Gesture)
- WD OFF with empty/None motions causes "sticky" properties that never reset
- Direct BlendTrees and Additive layers MUST always be WD ON (values fly off with WD OFF)
- Mixed WD in a controller causes random properties to stick
- **Rule:** All states WD ON. Never mix within a controller.

### Playable Layers

| Layer | Purpose |
|---|---|
| **Base** | Locomotion (humanoid muscles). Default supplied by VRChat. |
| **Additive** | Additive on top of Base (breathing, idle animations). Always WD ON. |
| **Gesture** | Hand gestures, face expressions. Masked to upper body / hands. |
| **Action** | Full-body overrides (emotes, AFK). Starts at weight 0; use VRCPlayableLayerControl to blend in. |
| **FX** | Non-transform only: toggles, blendshapes, materials, shader properties. |

### Toggle & Feature Inspection Workflow


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sentfromspacevr/vrchat-agentic-tools](https://github.com/sentfromspacevr/vrchat-agentic-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
