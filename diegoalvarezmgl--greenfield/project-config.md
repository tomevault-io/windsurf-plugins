---
trigger: always_on
description: You are in **GreenField**, a prescriptive build manual with copy-ready templates for shipping a
---

# AGENTS.md — GreenField

You are in **GreenField**, a prescriptive build manual with copy-ready templates for shipping a
Next.js web app (which is also the only backend) plus a native Kotlin and Compose Android client,
on two Firebase projects — staging and production — with Web and Android sharing the project for
their environment, in a pnpm and Turborepo monorepo.

**This repository is not a product.** Nothing here is deployed. The deliverable is the manual in
`vault/` and the templates in `vault/90-Templates/`, which are copied into new product repositories.

> This file governs work on **the manual itself**. It is not the `AGENTS.md` that products get —
> that one is a template at `vault/90-Templates/files/root/AGENTS.md.template` and describes a
> product's rules, not these.

## Repository map

```text
greenfield/
├── AGENTS.md            ← you are here
├── README.md            ← the public entry point
├── SECURITY.md          ← disclosure policy; a template flaw affects every product built from it
├── vault/               ← THE MANUAL. An Obsidian vault, 104 notes
│   ├── Home.md            index
│   ├── 00-…12-           standards, decisions
│   ├── 90-Templates/      manifest.json plus the literal files
│   └── 91-Runbooks/       ordered procedures
├── skills/              ← agent skills for operating on GreenField itself
└── tools/check-vault.mjs  integrity checks, also run in CI
```

Only `vault/` is copied into a product. Everything else belongs to this repository.

---

## If the user wants to start a new product

This is the most common request. Do this, in order:

1. Read `vault/00-Start-Here/Bootstrap-A-New-Product.md` — the master ordered recipe, which cites
   the exact note for every step.
2. Read `vault/91-Runbooks/Runbook-Bootstrap-New-Product.md` — the command-level procedure,
   including the `product.json` answers file.
3. Follow them. Do not improvise a step a note already specifies, and do not reconstruct the
   procedure from this file.

### Where the product goes

The product is **never** created inside this repository. There are two layouts, and which one
applies depends on how you got here:

| You are working in | The product goes | Then |
| --- | --- | --- |
| A clone made *inside* the product's own directory, i.e. `my-product/greenfield/` | `my-product/`, the parent | **Delete this clone** before `git init`. It is scaffolding |
| A permanent clone, e.g. `~/src/greenfield` | A sibling directory | Leave this clone alone |

The first is the normal case and the one the runbook documents. The clone is a delivery mechanism,
like a downloaded template: the manual is copied to `my-product/docs/` and the clone is discarded
before the product's history begins, so the nesting never reaches a commit.

Nothing is lost by discarding it. `docs/` holds the whole manual, and the product's own `AGENTS.md`
and `.cursor/skills/` are materialized from templates.

### The five things that go wrong here

| Mistake | Why it matters |
| --- | --- |
| `git init` in the product before removing the clone | A repository inside a repository is a rejected pattern (ADR-0001) and forces a gitignore workaround. Order: `rm -rf greenfield`, then `git init` |
| Copying more than `vault/` | This repository's README, licence, CI, `skills/` and `tools/` are the manual's, not the product's. Its CI checks paths that will not exist there and will fail |
| Losing the provenance record | Write `.greenfield-origin` with this clone's commit **before** deleting it. A GreenField advisory cannot reach a copy; that file is what tells the product whether one applies |
| Choosing the Android application id casually | Once an artifact is uploaded to Play under an id, that id can never be changed or reused |
| Skipping `--dry-run` on the materialize script | It reports unresolved placeholders before writing about fifty files |

### Human-only steps

Billing account linkage, Firebase terms acceptance, Play Console and payment provider account
creation, and the first manual bundle upload cannot be scripted. The runbook marks each one. Stop
and hand over rather than pretending they succeeded.

---

## If the user wants to change the manual

### The rules

- **A note and its template must agree.** A standard saying one thing while the template ships
  another is the worst defect this repository can have: the builder follows the template and
  believes the note. Fix both in one commit.
- **A note may describe a capability, never a product.** "A structured-output endpoint" is correct;
  naming a real product is a leak. No product names, domains, project ids or application ids.
- **Reversing a locked decision means superseding its ADR**, never editing an accepted one in
  place. Then update `vault/00-Start-Here/Decision-Register.md`.
- **Add every new note to the link graph.** An unlinked note is invisible, and the checks fail on
  orphans.
- **Versions live in exactly two notes**, the web and Android stack baselines, plus the templates
  that literally contain them. Do not repeat a version anywhere else.
- Bump `updated:` in the frontmatter of every note you touch.

### Adding a template

1. Put the file under `vault/90-Templates/files/**` with a `.template` suffix.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [diegoalvarezmgl/greenfield](https://github.com/diegoalvarezmgl/greenfield) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
