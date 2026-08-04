---
trigger: always_on
description: **The product is a drop-in `mgfxc` replacement: a self-contained library** a user adds to their **MonoGame/KNI project on Linux, macOS, or Windows**, that compiles **`.fx` → `.mgfx` in memory at runtime**, requiring **nothing but the library itself** — no `fxc.exe`, no `mgfxc`, no Wine, no Windows SDK, no native toolchain the user has to install separately. Its output **loads and renders identically to `mgfxc`'s** in the **real MonoGame/KNI runtime**. **One faithful compiler; the same `mgfxc`-eq
---

# ShadowDusk — Cross-Platform MonoGame Shader Compiler

## THE PURPOSE (read this first)

**The product is a drop-in `mgfxc` replacement: a self-contained library** a user adds to their **MonoGame/KNI project on Linux, macOS, or Windows**, that compiles **`.fx` → `.mgfx` in memory at runtime**, requiring **nothing but the library itself** — no `fxc.exe`, no `mgfxc`, no Wine, no Windows SDK, no native toolchain the user has to install separately. Its output **loads and renders identically to `mgfxc`'s** in the **real MonoGame/KNI runtime**. **One faithful compiler; the same `mgfxc`-equivalent result everywhere.**

The distinctions that carry the most weight — internalize these, they have drifted before (full detail, success criteria, and the backend table in **[docs/the-purpose.md](docs/the-purpose.md)**):

- **The library *is* the product.** The deliverable is the in-memory compiler called at runtime (`IShaderCompiler.CompileAsync(fx) → .mgfx bytes`). The **CLI** and **MGCB plugin** are *delivery shapes of the same library*; the **browser / WASM shader-fiddle is ONLY a sample of reach — never the product.** Don't let sample work redefine the goal.
- **One pipeline, everywhere — NO substitute compilers.** Every host runs the same faithful pipeline (HLSL →`[DXC]`→ SPIR-V →`[SPIRV-Cross]`→ GLSL →`[managed rewrite + MGFX writer]`→ `.mgfx`; or `vkd3d-shader` → DXBC for DirectX). A host must **not** swap in a different frontend/compiler to make a platform "work" — different compiler ⇒ different output ⇒ silently breaks the "identical to `mgfxc`" promise. If a faithful component can't run on a host yet, that host's runtime-compile is **not done** — never a licence to substitute.
- **"Self-contained" is a hard requirement.** Native pieces ride *inside* the NuGet package (transitive native assets), never a separate manual install. "Add the package, call the API" is the entire setup.
- **The bar is the real runtime, not our tests.** Only ShadowDusk's `.mgfx` loading in MonoGame's `Effect` and rendering like `mgfxc`'s proves the promise. Tests/our-own-renderer images are **proxies, not the bar**. Compare same-backend only (GL↔GL, DX↔DX), never cross-backend. "Same as `mgfxc`" = behaviorally equivalent + `Effect`-loadable, **NOT** byte-identical (that's a non-goal).

### The evidence ladder — what "rung 4" means

Docs, phase notes, and commit messages say **"rung 4"** constantly. It is the four-step scale of how strongly a target is proven, weakest to strongest:

| Rung | What it proves |
|---|---|
| **1** | The shader **compiles** without error. |
| **2** | The output file is **structurally well-formed** (a real reader can parse it). |
| **3** | Our output **matches the reference compiler's** (`mgfxc`/`fxc`) when rendered in **our own** test renderer. |
| **4** | The output **loads in the real engine** (MonoGame/KNI `Effect`, or FNA) and **renders the same image** as the reference compiler's build. |

**Only rung 4 proves the promise.** Rungs 1-3 are proxies: every one of them can be green while the product is broken for a real player. "Rung-4 proven" and "render-proven" mean the same thing.

## Source-of-truth files

- **[project_facts.md](project_facts.md)** — what is TRUE (targets and how far each is proven, pins and natives, where things run, known gaps, vocabulary).
- **[project_rules.md](project_rules.md)** — how to WORK on it (testing bar, code conventions, docs/phase process, release mechanics).
- **[project_decisions.md](project_decisions.md)** — what was CHOSEN and why; consult before re-litigating anything.

**Do NOT create memory files, and do NOT rely on the machine-local agent memory store** (it is lost between computers). Every durable fact, rule, or decision goes in those three files — exceptions that stay separate: phase docs, user-facing docs/readmes, reference docs, temp files. Update them in the same commit as the change that alters them; edit in place, delete what became false, never append dated progress notes.

The rules below stay in this file *because they must fire without anyone opening another file*. They are not duplicated in `project_rules.md`.

### Handoff — there is no verbal handoff on this project

Work arrives from a previous session you did not see. Two obligations, both non-optional:

- **Picking up:** before assuming the state of anything, read `git log --oneline -20`, `CHANGELOG.md`'s `[Unreleased]` section, and `plan/plan.md`'s status rows.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kaltinril/ShadowDusk](https://github.com/kaltinril/ShadowDusk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
