---
trigger: always_on
description: Single source of truth for agents working on **abap2UI5 Samples** — a collection
---

# AGENTS.md

Single source of truth for agents working on **abap2UI5 Samples** — a collection
of demo apps for the abap2UI5 framework. This file owns everything: the folder
scheme, the compatibility model, the overview generation rules, **and** the
ABAP code style / app-structure conventions.

> These instructions OVERRIDE any default behavior and must be followed exactly.

## Language

**This entire project is in English.** All code, comments, commit messages, PR
titles, PR descriptions, and any other text must be written in English.

## Pull requests

- **The PR title becomes the squash-merge commit subject — make it describe
  the change.** Before merging, replace any auto-generated title (e.g. a
  branch name like `Claude/...-abc123`) with a short descriptive English
  title that states what actually changed.
- **One topic per PR.** A structural change (moving, adding, or renaming
  subpackages) must not ride along in a PR titled for an unrelated sample —
  split it into its own PR so the history stays searchable.

---

## 1. Repository layout

All samples live under `src/`, split into exactly two top-level packages
(abapGit `FOLDER_LOGIC=PREFIX`, `STARTING_FOLDER=/src/`). There are **no demo
apps directly in `src/` root** — every sample sits in a categorised subpackage.

```
src/
├── 01/  "basic"     cloud-ready & downportable — survives every build
│   ├── 01/  Basic I
│   ├── 02/  Basic II     framework actions, custom controls and use cases
│   └── 08/  Control Library     1:1 rebuilds of UI5 demo kit samples, split by library
│       ├── 00/  controls - sap.m
│       ├── 01/  controls - sap.uxap
│       ├── 02/  controls - sap.f
│       ├── 03/  controls - sap.ui.core
│       ├── 04/  controls - sap.ui.layout
│       ├── 05/  controls - sap.tnt
│       ├── 06/  controls - sap.ui.codeeditor
│       └── 07/  controls - sap.ui.unified
└── 00/  "extended"  restricted / special-purpose — STRIPPED from cloud & 702 builds
    ├── 00/  extended                     restricted samples without a more specific category
    ├── 01/  only non-abap-cloud          on-premise-only ABAP (not ABAP Cloud ready)
    ├── 02/  only non-openui5 or higher UI5 1.71   SAPUI5-only controls (sap.suite.*, sap.ui.comp.*, VizFrame, …) or a control/property introduced after UI5 1.71
    ├── 03/  only with launchpad          runs only inside the Fiori Launchpad
    ├── 05/  only with javascript and css and html   needs native JS / CSS / HTML
    ├── 06/  only testing                 test / scaffolding apps, not demos
    ├── 07/  experimental, TODO           work-in-progress / not finished
    └── 99/  obsolete                     superseded, or built on a deprecated UI5 control
```

This tree is machine-checked: `node scripts/check-agents-structure.js` compares
it against the actual `package.devc.xml` `<CTEXT>` values and fails on any
drift (runs in CI). **Whenever a subpackage is added, removed, or renamed,
update this tree in the same change.**

Each subpackage's `package.devc.xml` `<CTEXT>` is the human-readable name shown
above (e.g. `only non-abap-cloud`). **That CTEXT string is also the overview
group name — keep the two identical** (see §4).

> Class names never encode the folder (`FOLDER_LOGIC=PREFIX`). Moving a sample
> between packages needs **no rename** and keeps navigation intact — but the
> overview catalog must be updated (§4).

Every sample in `01/08` is a faithful rebuild of one specific UI5 demo kit
sample, filed in the subpackage of the library its entity belongs to
(`01/08/00` = sap.m, `01/08/01` = sap.uxap, …), and carries the demo kit URL
as an ABAP Doc line directly above its `CLASS ... DEFINITION`
(`"! Rebuild of the UI5 demo kit sample: <url>`).
Its `<DESCRIPT>` follows the convention `<entity> - <demo kit description>`
(e.g. `sap.m.Switch - "Some say it is only a switch, I say it i`), where the
entity is the control from the demo kit URL and the description comes from
the library's `demokit/docuindex.json` in openui5 (HTML markup stripped,
truncated to the 60-character DESCRIPT limit). The **full, untruncated**
description is kept as additional ABAP Doc lines below the URL line; the
overview generator prefers those lines as the tile `sub` (§4).
Demos that have no demo kit original do not belong in `01/08` — file them in
the framework package (`01/02`, actions / custom controls / use cases) or,
when a restriction applies, in the matching `src/00` category.

Machine-generated demo kit ports that have not been manually reviewed do not
live in this repository — they are collected in the separate api repository
of the abap2UI5 organization. Everything under `01/08` here is manually
reviewed.

---

## 2. Compatibility model — what belongs in `src/01` vs `src/00`

The split is driven directly by the CI builds:

| Build (workflow)   | What it does                                    | Sees `src/01` | Sees `src/00` |
|--------------------|-------------------------------------------------|:---:|:---:|
| `ABAP_STANDARD`    | `abaplint ./abaplint.jsonc` (syntax `v750`)     | ✅ | ✅ |
| `ABAP_CLOUD`       | `rm -r src/00` → `abaplint abap_cloud.jsonc`    | ✅ | ❌ |
| `ABAP_702`         | `npm run downport` (does `rm -rf src/00`) → `abaplint abap_702.jsonc` | ✅ | ❌ |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [abap2UI5/samples](https://github.com/abap2UI5/samples) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
