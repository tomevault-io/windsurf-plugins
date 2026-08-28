---
trigger: always_on
description: This is the **kit itself** — a method for building marketing sites, not a site. If you are
---

# Working on this repo

This is the **kit itself** — a method for building marketing sites, not a site. If you are
building a site, you want the `website-build` skill; this file is about maintaining it.

## What lives where

```
skills/website-build/SKILL.md         short. what to do, in order
skills/website-build/references/      the detail, loaded only when needed
  kickoff.md    source import, discovery rounds, feature catalogue, design tokens, mobile
  stacks.md     migration playbook per builder; integration inventory; every provider + default
  archetypes.md page shape per site type — section order, proof model, failure mode
  features.md   404, search, light/dark/auto, i18n, shortcuts — decisions with a shape
  design.md     full redesign — the comp process, expensive vs templated, the tells
  build.md      standing instructions, phases with gates, definition of done
  compliance.md which accessibility law binds this client; what to build, test, publish
  traps.md      silent failures, with symptom and fix
commands/       slash-command entry point
template/       Astro + Cloudflare starter — must build green from a clean clone
docs/           for humans using or extending the kit
```

`SKILL.md` stays short. Detail goes in a reference — the point of the split is that the
model loads what it needs, not everything.

## The bar for each kind of change

**A trap** — it failed **silently** on a real build. Clean build, clean types, clean deploy,
wrong result. Not "this is good practice". If a compiler, linter or obvious error message
would have caught it, it does not belong.

**A provider or stack** — needs three things: when you would pick it, what picking it costs,
and how it compares to the default. Do not add something you have not shipped.

**An archetype** — section order, proof model, where conversion sits, and the failure mode. All
four, or it is a description rather than a decision. It describes **structure, never
appearance** — the moment it specifies how something looks, it belongs in the design direction
in `kickoff.md`, and the template stops being a skeleton.

**A design entry** — process only if it changes the outcome; tells only if they are checkable
by looking at a page. It explains why something reads as expensive and never prescribes a look —
name a hex and it has become a design system.

**A feature entry** — only where the *yes* has a shape: a decision that costs a rebuild if
taken wrongly. Needs the default, the condition that changes it, and the specific failure.
Nothing in `features.md` ships in the template, for the same reason the template has no design.

**A compliance entry** — `compliance.md` §1 dates and thresholds need a source link and a checked-on date, or
they do not go in. This is the only file that goes stale without anyone touching it; two US
deadlines moved a full year in 2026. `compliance.md` §5 and §8 take entries on the same terms as a trap: it
failed on a real build. Do not transcribe the WCAG spec — it exists and it is better.

**A template addition** — would you write this from scratch next project, and would you get
it wrong the first time? Media pipeline yes. Hero layout no.

The template is a skeleton, not a theme. If it accumulates opinions about how a page should
*look*, every site built from it starts looking the same. Two hard tests before anything
lands in `template/`:

- **Could the template render a page that looks finished?** If yes, it has a design and the
  next project inherits it. It ships with a grey placeholder ramp, no typeface, no home page
  and no hero treatment; `npm run tells` enforces that they are cleared together
- **Is the entry structure, behaviour or plumbing?** A focus trap, a `dvh` panel, a busy state,
  the KV-before-provider ordering — those are things you would get wrong. A card style, a
  gradient, a nav hover, an icon depicting the trade — those are things you would *choose*

`global.css` carries interactive **states**, never a look. `.btn` exists so nobody ships a
control missing `:disabled`; the moment it grows a pill radius and a hover lift, it is a theme.

**Provenance stays out of `template/`.** No colour sampled from a specific client's old theme,
no client's typeface, no trade-specific icon set, no storage key or phone number from a real
build — including inside a JavaScript error string, which is where one hid for two projects.
Name the build a *lesson* came from in the skill references; never leave its artefacts in the
starter.

## After changing the template

```bash
cd template && rm -rf node_modules dist .astro && npm install && CI=true npm run build:staging
PUBLIC_SITE_ENV=staging npm run check             # 0 errors
npm run tells                                     # "fresh template", tells under three
npm run check:sitemap                             # no-op on staging, must still exit 0
```

Must build green with **no content, no images and no secrets**. That is the promise; a
starter that needs setup before it compiles is not a starter.

`npm run tells` on a clean clone must report **"fresh template"** — all placeholders present,
none cleared. A half-cleared state means something ships a design decision it should not.

### Then the provenance sweep


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nurkamol/website-build-kit](https://github.com/nurkamol/website-build-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
