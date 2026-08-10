---
trigger: always_on
description: **This is the canonical, enforceable process doc for this repo — read it first.** It applies to every
---

# CLAUDE.md

**This is the canonical, enforceable process doc for this repo — read it first.** It applies to every
agent (Claude Code, Codex, and sub-agents). `GODOT_BEST_PRACTICES.md` is its companion and the
canonical source for Godot-specific design, runtime, testing, validation, and harness-invocation
rules. `AGENTS.md` simply points here. Keep process rules in this file (and Godot specifics in
`GODOT_BEST_PRACTICES.md`) so they don't drift across docs.

## Branch & worktree workflow (DEFAULT)

- **`0.3-dev` is the main development branch** — the integration branch all feature work targets.
  `main` is downstream. Do **not** commit feature work directly to `main`.
- **Do every non-trivial change in a dedicated git worktree branched off `0.3-dev`**, not in the
  primary checkout:
  `git worktree add ../local-agents-<feature> -b feature/<name> 0.3-dev`
  Build there, commit as you go, and merge back into `0.3-dev` only when verified. This is the
  standard because another session/agent running git ops (checkout/reset/merge) on the shared
  checkout has corrupted and wiped untracked in-progress work here before — an isolated worktree
  makes your files immune to another writer's branch switches.
- The compiled GDExtension `bin/` is a gitignored build artifact absent from a fresh worktree —
  symlink it from the primary checkout so the extension loads:
  `ln -s <primary>/addons/local_agents/gdextensions/localagents/bin <worktree>/addons/local_agents/gdextensions/localagents/bin`
- When a feature is verified, merge it into `0.3-dev`, then prune: `git worktree remove <dir>` and
  `git branch -d feature/<name>` (delete the pushed remote branch too once merged).
- Skip the worktree only for trivial single-file edits (docs) or when you have confirmed you are the
  sole writer. **Never** run a bulk-edit sub-agent on files you (or another lane) are also
  live-editing; commit before any bulk delete so a mistake is one `git checkout` away.

## 3D assets: convert FBX → glTF (DEFAULT)

- **Godot renders glTF (`.glb`/`.gltf`) reliably; FBX is the fragile path.** Bring every 3D asset in as
  **glTF**. **Do not** rely on Godot's ufbx FBX importer for skinned/animated meshes at runtime.
- **Non-skinned FBX (caps, hair, props):** Godot itself is the converter — `GLTFDocument.append_from_scene`
  + `write_to_filesystem`, headless. Fine for static/rigid meshes.
- **Skinned/animated characters: convert with headless Blender** — Godot's own FBX→glTF path left the
  skinned Kenney character **invisible** (a ufbx/skin quirk), so use Blender's exporter, which produces a
  clean, upright, Godot-friendly `.glb`. Worked example: **`blender_convert_female.py`**
  (`/Applications/Blender.app/Contents/MacOS/Blender --background --python <script>`). It:
  - imports the character mesh FBX + the separate idle-animation FBX (Kenney ships animations as their
    own files);
  - picks the real **Idle** action (idle.fbx also carries a "0_Targeting Pose" that raises the arms —
    grab the one whose name has `idle` and not `target`);
  - **re-binds the mesh to the idle armature** (re-point the Armature modifier + reparent) instead of
    cross-assigning the action — cross-assigning across two armatures breaks when their rest poses
    differ (symptom: body **bobs but arms stay in a T-pose**);
  - paints the skin as a Principled BSDF base-color texture and exports one `.glb` (`export_yup=True`).
- **Runtime gotchas seen:** the Blender clip imports as a compound name like `Root_001|Root|Idle` (match
  by substring, don't hardcode `"Idle"`); set the clip `loop_mode = LOOP_LINEAR` or it one-shots; the
  character may face +Z (add a 180° yaw). **Attach head accessories with `BoneAttachment3D`** bound to
  the `Head` bone so they track the skeletal idle + gaze through the node tree — no per-frame sync.

## Destructive-command safety (bulk delete/find)

Do **not** delete files with `find ... -name <dir> -exec rm -rf` or a bare recursive `rm` that walks
`scenes/simulation/`. The live `voxel/` subtree has `actors/`, `ui/`, and `shaders/` subdirectories
whose **names collide** with old-stack siblings, so a name-based `find` silently matches the new
scene too (this already nuked `voxel/{actors,ui,shaders}` once — recovered only because it was
committed). When removing files:
- Prefer **explicit paths** or `git rm <path>` (it refuses to touch untracked files and stages the delete for review).
- If you must `find`, scope it: anchor with `-path '.../scenes/simulation/actors'` (full path, not `-name`),
  or add `-maxdepth 1`, and never combine `-name` with `-exec rm`/`-delete` over a shared parent.

## Execution model

- Prefer planning before large changes: understand current state and risks before editing; for big or
  ambiguous work start with a short investigation pass.
- The main thread MAY perform implementation edits itself — it is **not** limited to orchestration, and
  there is no rule that all implementation must be delegated. **But editing is permitted only inside its
  OWN dedicated worktree off `0.3-dev`, NEVER directly on the shared `0.3-dev` primary checkout.** The
  distinction is exact: the main thread may *edit*; it may not edit/commit on the shared main-branch

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [adammikulis/local-agents](https://github.com/adammikulis/local-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
