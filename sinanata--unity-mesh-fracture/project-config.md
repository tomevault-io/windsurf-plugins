---
trigger: always_on
description: Guidance for AI coding agents (Codex, Cursor, GitHub Copilot, Claude Code, Windsurf, Aider, Zed, and others) working in this repository or wiring this tool into another project. Humans: this is a fast, accurate map. Deeper docs are linked at the bottom.
---

# AGENTS.md

Guidance for AI coding agents (Codex, Cursor, GitHub Copilot, Claude Code, Windsurf, Aider, Zed, and others) working in this repository or wiring this tool into another project. Humans: this is a fast, accurate map. Deeper docs are linked at the bottom.

## What this project is

A drop-in **Voronoi mesh fracturer for Unity 6 + URP**. Pure-C# fragmentation (zero dependencies beyond `UnityEngine`) that pre-bakes watertight two-submesh fragments — with cooked convex hulls — at load time, then plays them as a fade-out debris burst with optional Unity physics. The entire runtime is three files under `Assets/MeshFracture/Runtime/`. Namespace: `MeshFracture`. License: MIT. Battle-tested in the shipping cross-platform game Leap of Legends, where every character that explodes runs through this pipeline.

## Golden rules (do not violate)

1. **"Burst" here is the explosion, not Unity's Burst compiler.** `FractureBurst` is a plain `MonoBehaviour`; the whole runtime is single-threaded managed C# over `UnityEngine` types (`Mesh`, `Vector3`, `List<T>`, `Quaternion`). Do **not** add `[BurstCompile]`, `IJob*`, `NativeArray<T>`, `Unity.Mathematics.float3`, or `using Unity.Collections / Unity.Burst / Unity.Jobs`. None are dependencies; none are used. Pattern-matching "Burst mesh fracture" onto the DOTS/Jobs stack is the single most common wrong turn here.
2. **Everything runs on the main thread; bake at load, look up at impact.** `MeshFragmenter.Fragment` is a synchronous, O(n²)-in-fragment-count CPU call. Never call it per-impact during gameplay. Use `FragmentCache.RequestPreBake` (async, one bake per frame) or `FragmentCache.BakeSynchronous` (blocking, loading-screen only) at load time, then `FragmentCache.TryGet` at the moment of impact. `TryGet` returning `false` means "not baked yet" — fall back to a particle/smoke puff, don't block.
3. **Source meshes must be Read/Write-enabled.** `Fragment` reads `source.vertices / normals / uv / triangles`; on a non-readable imported mesh those come back empty and you get a single-fragment fallback. Enable Read/Write on the model importer (the demo's `DemoModelImporter` does this for its FBXes).
4. **Fragment materials must be transparent and owned per-burst.** `Initialize` animates `_BaseColor.a` 1 → 0 by mutating the material instance directly (MaterialPropertyBlock overrides are silently dropped on some URP / SRP-Batcher / WebGL combinations). Hand each burst its own `Material` instances (`new Material(interiorMat)`), configured transparent + Cull Off + **ZWrite ON** — copy `MaterialFactory.ConfigureFractureTransparent`. ZWrite must stay ON, or solid chunks render see-through-to-interior.
5. **Two submeshes per fragment:** submesh 0 = exterior (the original textured surface), submesh 1 = cap/interior (raw cut faces). Always pass both an exterior and an interior material to `Initialize`.
6. **No custom `.shader` assets.** The runtime ships zero `.shader` and zero `.mat` files on purpose — a hand-rolled URP shader rendered every fragment solid black on WebGL2 (an HLSL → GLSL ES 3.0 cross-compile bug). Stock URP/Lit configured at runtime sidesteps it. Do not reintroduce a custom shader.
7. **No `using LeapOfLegends.*` or other product-specific imports.** New runtime code lives in the `MeshFracture` namespace only; demo-only code lives under `MeshFractureDemo`.

## Wire it into your project

The runtime is one folder — `Assets/MeshFracture/` — dropped into your `Assets/`. The full gameplay wiring is:

```csharp
using MeshFracture;

// 1. At load (a loading screen), pre-bake. Voronoi is ~3-9 ms/character on desktop.
int key = prefab.GetInstanceID() ^ (fragmentCount * 7919);
FragmentCache.RequestPreBake(key, prefab, fragmentCount);

// 2. At impact, look up and spawn the burst.
if (!FragmentCache.TryGet(key, out var cached)) return;   // not baked yet -> particle fallback
Vector3 offset = deathPos - cached.MeshCenter;
var fragments = new FragmentResult[cached.Meshes.Length];
for (int i = 0; i < fragments.Length; i++)
    fragments[i] = new FragmentResult { Mesh = cached.Meshes[i], Centroid = cached.LocalCentroids[i] + offset };

var burst = new GameObject("FractureBurst").AddComponent<FractureBurst>();
burst.transform.position = deathPos;
burst.Initialize(fragments, exteriorMat, new Material(interiorMat),
                 explosionForce: 18f, meshRotation: cached.MeshRotation);
```

For bouncing, collision-aware chunks, set these before `Initialize`: `burst.UseUnityPhysics = true; burst.PreBakedColliderMeshes = cached.ColliderMeshes; burst.DissolveAfterSettle = true;`. The canonical minimal example is `Assets/MeshFracture/Demo/MeshFractureDemo.cs` (~140 lines, drop-on-a-cube).

## Public API

`MeshFragmenter` (static — `Assets/MeshFracture/Runtime/MeshFragmenter.cs`):
```csharp
static FragmentResult[] Fragment(Mesh source, int count, Vector3 center);   // deterministic from (source, center)
static Mesh BakeSkinnedMesh(SkinnedMeshRenderer smr);                       // SMR-local space; BakeMesh(useScale:false) — do not double-scale

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sinanata/unity-mesh-fracture](https://github.com/sinanata/unity-mesh-fracture) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
