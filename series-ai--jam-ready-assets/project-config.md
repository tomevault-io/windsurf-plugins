---
trigger: always_on
description: > ## ⚠️ This repository is PUBLIC. Everything in it is public communication.
---

# AGENTS.md - AI guide to this asset library

> ## ⚠️ This repository is PUBLIC. Everything in it is public communication.
>
> `series-ai/jam-ready-assets` is open to the world. Anyone, logged in or not, can read every
> file, commit message, pull request title and body, issue, and code review comment, and search
> engines index them. Treat anything you write here as published under the RUN name, because it is.
>
> **Write for an outside reader, and assume creators whose work is in the library will read it.**
>
> - **Never name a creator or pack in a negative light.** Do not record that a pack was rejected on
>   taste, that a creator's licence wording is contradictory, or that their art was not good enough.
>   Make the technical point without the name: "a store licence tag can disagree with the page terms"
>   rather than naming who. A rejection note that helps nobody here can cost a small creator real
>   standing, and they can find it.
> - **Never put a personal name in a committed file.** `Verified-by` is `run-workshop maintainers`,
>   not a person. These licence files are copied into every creator's project, so a name here travels
>   much further than a commit author line.
> - **No internal references.** No Linear issue keys (`RUN-123`), no references to private repos or
>   their pull requests, no internal service names, dashboards, Slack channels, or employee emails.
>   Describe the reason, not the ticket.
> - **No internal process narration.** Sprint plans, review gates, launch dates, headcount, and
>   anything about what the company is about to ship stay out.
> - **Keep the licence reasoning, drop the gossip.** Explaining *why* a pack is admissible is useful
>   to everyone and belongs here. Explaining *who* failed and how is neither.
> - **Commit messages are the hard case.** A pull request body can be edited; a commit message that
>   has been pushed is effectively permanent. Get it right the first time.
>
> If something genuinely needs to be said and cannot be said in public, put it in the internal
> tracker and link nothing.

This repo is a curated library of **free game art** for building games on the **RUN platform**. This file tells an AI agent how to find, choose, and use assets here, and how to add more without breaking the structure.

## TL;DR for agents

- Everything here is **free to use, modify, and redistribute in any RUN game**, commercially included. There is no single library-wide licence: each pack carries its own `License.txt`, and CI refuses any licence outside **CC0-1.0, MIT, BSD-2-Clause, and the RUN License** (`LicenseRef-RUN-Repository-Supplemental-1.0`, run-workshop's `LICENSE.md`). Of 338 packs, 318 are CC0, 4 are MIT (Proof of Play *Pirate Nation*), and 16 are under the RUN License (every RUN Voxel Packs leaf: RUN projects only until it converts to MIT on 2028-01-01; the `3D/characters` leaves also keep Proof of Play's MIT notice for the Pirate Nation rig data they carry).
- **Never delete or move a pack's `License.txt`.** For CC0 packs it is provenance; for the others it is the whole obligation, and RUN.studio copies it into the creator's project alongside the assets.
- The public schema-v1 manifest stays **CC0-only** for older Studio builds. Licence-aware consumers use `manifest/v2/`, which may include MIT, BSD-2-Clause and RUN License packs.
- Layout is **predictable and pack-first**: `<creator>-<pack>/<dimension>/<theme>/` (plus flat `<pack>/ui|icons|fonts|audio/` buckets). Find assets by globbing themes across packs (`ls -d */2D/platformer`), not by guessing filenames.
- For 3D in a RUN game, **use the `.glb`/`.gltf` file** in a pack (the RUN runtime loads it directly). `.fbx`/`.obj` are editable source only.
- Files are stored with **Git LFS**. After cloning, run `git lfs install && git lfs pull` or you'll only see pointer text, not real assets.

## Repository layout

Every pack is a top-level directory. Inside it, content is bucketed by dimension
(with a theme level under `2D/` and `3D/`) or by flat type buckets:

```
<creator>-<pack>/2D/<theme>/     pixel art, sprites, tilesets, spritesheets (.png, .aseprite)
<creator>-<pack>/3D/<theme>/     models & kits (.glb, .gltf, .fbx, .obj, textures)
<creator>-<pack>/ui/             buttons, panels, cursors, HUD frames
<creator>-<pack>/icons/          game icons, input-prompt icons
<creator>-<pack>/audio/          music, SFX, voice (.ogg, .wav, .mp3)
<creator>-<pack>/fonts/          bitmap & web fonts
```

Most packs hold a single bucket, but one pack may span several, e.g.
`proofofplay-pirate-nation/` carries `3D/pirate/`, `ui/`, `icons/`, and
`audio/` in one place. Each bucket/theme leaf is one catalog pack in the
manifest, with id `<pack>/<bucket>[/<theme>]`.

- The **creator prefix** on every pack folder (`kenney-`, `kaykit-`, `pixel-frog`/`grafxkid-`, etc.) tells you the source at a glance.
- Every pack carries a `License.txt` in its root naming its licence and the source it was verified against. Any original licence or readme the pack shipped with is kept alongside it.
- Full creator credits + source links are in [`README.md`](README.md).

## Themes


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [series-ai/jam-ready-assets](https://github.com/series-ai/jam-ready-assets) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
