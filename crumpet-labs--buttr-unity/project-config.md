---
trigger: always_on
description: Buttr is a dependency-injection container first, and the container has no opinion about how you name or organise your code — it works with whatever conventions you already use. What follows is a *separate*, optional layer: an architecture I've leaned on across real projects because it pays for itself.
---

# Conventions

Buttr is a dependency-injection container first, and the container has no opinion about how you name or organise your code — it works with whatever conventions you already use. What follows is a *separate*, optional layer: an architecture I've leaned on across real projects because it pays for itself.

The idea is simple: every type carries a **suffix that says what it does** — `Service`, `View`, `Registry`, `Handler`. That single rule buys two things worth more than they look:

- **It kills analysis paralysis.** You stop staring at a new file wondering where it belongs or what to call it. The front door other features call through? A `Service`. Something that reads the model and renders it? A `View`. The suffix decides, so your attention goes to the code instead of the shape of the folder.
- **It scales to a team.** Anyone can open a feature they've never seen and read its structure at a glance. Onboarding, review, and handoffs stop leaning on tribal knowledge, because the shape is legible on its own.

None of it is required, and you don't adopt it all at once — reach for the handful of suffixes a feature actually needs (`Service`, `Model`, `View`, `Controller`) and pull in the rest when a problem calls for them. The table below is the full vocabulary, not a checklist.

For a quick tour, see [Getting Started](GettingStarted.md).

---

## Project structure

Buttr organises by feature, not by file type. Everything lives under `Assets/_Project/`.

```
Assets/_Project/
├── {ProjectName}.asmdef
├── Main.unity                  # Boot scene
├── Program.cs                  # Application composition entry point
├── ProgramLoader.cs + .asset   # Bridge to Unity's boot lifecycle
├── Core/                       # Game-agnostic packages — travels across projects
├── Features/                   # Game-specific feature packages
├── Shared/                     # Assets and scripts used by both Core and Features
└── Catalog/                    # ScriptableObject data assets, organised by feature
```

**Core** is your reusable library: logging, event systems, save systems, audio management. It's game-agnostic and rarely changes. **Features** are game-specific: inventory, combat, dialogue, crafting. They're built for the game in front of you right now.

Features contain code and feature-specific resources. **All ScriptableObject assets** live in `Catalog/`, mirroring the structure of `Core/` and `Features/`. The script that defines a ScriptableObject lives in the feature; the `.asset` instance lives in `Catalog/`. Loader assets are the one exception — they live in the package's own `Loaders/` folder next to the loader script.

---

## General rules

**Seal your classes.** All classes should be `sealed` unless explicitly designed for inheritance. This communicates intent, prevents unintended inheritance, and allows the compiler to optimise. If a class isn't `sealed`, it should be `abstract`.

**Minimise MonoBehaviours.** If something doesn't need Unity lifecycle hooks or `GetComponent<T>` access, it should be a plain C# class that gets injected. Don't make something a MonoBehaviour just because it's convenient.

**Scope.** Buttr handles feature-level architecture — singleton services, mediators, models, and registries that live once per feature. Per-instance entity management (hundreds of enemies, projectiles, spawned objects each with their own state) is a different domain that lives within the System and Controller layer, using whatever approach fits the project's performance needs. Buttr provides the structure those systems plug into, but doesn't prescribe how individual instances are managed.

---

## Suffix reference

| Layer | Suffix | What it does | Type |
|-------|--------|--------------|------|
| **Unity** | Controller | Coordinates Unity components and systems on a GameObject | MonoBehaviour |
| | Instance | Manages a scoped container's lifecycle on a GameObject (e.g. UI panels) | MonoBehaviour |
| **Data** | Model | Data — no behaviour, no dependencies | Class / Struct |
| | Id | Readonly struct for domain-driven identity | Struct |
| | Definition | ScriptableObject entry point into a feature | ScriptableObject |
| | Configuration | ScriptableObject providing editable settings | ScriptableObject |
| **Logic** | View | Observes and displays Model data — reads only | Class |
| | System | Reads Model state and executes it continuously | Class |
| | Mediator | Listens to events, filters and routes towards Services | Class |
| | Handler | Stateless ScriptableObject — designer-facing logic; pairs with a Definition for data | ScriptableObject |
| | Profile | Self-contained ScriptableObject bundling data and its own interpretation logic | ScriptableObject |
| | Behaviour | Stateful strategy — code-facing, drives a System's update loop | Class |
| **Infrastructure** | Service | Public API of a feature and its write-owner — the only type that mutates Model state; the entry point other features inject | Class |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Crumpet-Labs/Buttr.Unity](https://github.com/Crumpet-Labs/Buttr.Unity) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
