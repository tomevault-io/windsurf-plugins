---
trigger: always_on
description: Guidance for AI coding agents working on this repository. Human contributors
---

# AGENTS.md

Guidance for AI coding agents working on this repository. Human contributors
are welcome to read it too - it is the short version of how this card is put
together and what breaks it.

## What this is

`mini-humidifier` is a custom Lovelace card for Home Assistant: a single
JavaScript bundle that Home Assistant loads in the browser. There is no server
side, no Python, and no Home Assistant integration in this repository. The card
reads entity state out of the `hass` object and calls Home Assistant services
back.

Distribution is HACS: users get `mini-humidifier-bundle.js` as a release asset,
so **the release asset is the product**. A change is not shipped until it is in
a tagged release.

## The bundled models are presets, not the product

What this card is, is a kit: every control can be described in YAML - which
indicators exist and what each one reads, which buttons appear, what icon each
takes, what service each one calls, when each is disabled.
`src/configurations/` holds a dozen devices; the market has hundreds, and the
card is built this way so that a humidifier nobody here has heard of can go on a
dashboard without a pull request and without waiting for one.

That is a constraint on what may be changed, not just a description of what is:

- **A `model:` the registry does not know is supported.** It starts from the
  default configuration and warns in the console. Do not turn it into an error.
  [#112](https://github.com/artem-sedykh/mini-humidifier/issues/112) is a
  complete configuration for a `deerma.humidifier.jsq2w` - written by a user,
  posted with a screenshot, copied since - and refusing unknown ids would break
  it and every card copied from it.
- **The configuration is open at the leaves.** Indicator and button ids are
  chosen by the user, and their options are templates the user wrote. Anything
  that validates a configuration
  ([#178](https://github.com/artem-sedykh/mini-humidifier/issues/178)) or edits
  one ([#179](https://github.com/artem-sedykh/mini-humidifier/issues/179)) has
  to carry through what it does not recognise rather than dropping it. An editor
  that round-trips a hand-written card through a form and silently loses the
  half it does not model is worse than no editor.
- **Those templates are source text, not functions.** That is what forces
  several of the build constraints below - see "Why the model configurations
  stay JavaScript".

### How a preset is supposed to get into the bundle

The registry is meant to fill up from the outside, and it has: three of the
fourteen entries in `docs/models.md` are credited to the users who wrote them
(`by @ravikwow`, `by @regevbr`, `by @akovovh`). The intended path is one loop:

1. someone with a device the card does not ship for writes the configuration in
   their own YAML, because that always works;
2. if the device is common enough to be worth carrying, that configuration
   comes back as a pull request against `src/configurations/`;
3. it becomes a preset, and the next owner of that device gets it for free.

Which makes the second step worth watching for. `docs/models.md` documents it
under "Adding a model", but at the bottom of a page reached mostly by people who
already know unknown models work.
[#112](https://github.com/artem-sedykh/mini-humidifier/issues/112) is exactly
that step stalling: a complete configuration for a `deerma.humidifier.jsq2w`,
posted as an issue titled as a thank-you note in 2023, never turned into a
model. When one of those turns up, it is a contribution waiting for an invitation.

### The cost

The design is invisible. Someone whose device is not in `docs/models.md` reads
that table as a compatibility list and opens a model request; several of the
open issues are exactly that, from people who could have written the
configuration themselves in an evening and might have contributed it back.
Making this easier to find is worth more than bundling another model by hand.

## Helping someone with a device the card has no preset for

A different job from fixing a bug in `src/`, with a different output: the
answer is a YAML block, not a diff. Most requests that arrive as "add support
for X" are this one.

**The page to work from is [docs/custom-device.md](docs/custom-device.md).** It
carries the worked example (a Levoit Classic 300S through VeSync, from #124) and
the contract underneath the options - what a `source` is, what a template runs
as, which options are not templates, and what the card warns about rather than
refusing. Do not re-derive any of that from `src/`; if something there is wrong
or missing, the fix belongs in that page, where the next person also gets it.

What to establish before writing YAML, in this order:

1. **The entity's domain and attributes**, from Developer tools -> States. The
   bug report template already asks for these. Which attribute holds the target
   and which holds the reading is the fork everything else hangs off.
2. **The other entities the integration created** beside the humidifier - a
   night light, a switch, a sensor. Most "the card cannot do X" turns out to be
   X living on a second entity, which `state: entity` and `source: entity`
   reach.
3. **Which base**: `model: humidifier` for a `humidifier` entity, a bundled id

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [artem-sedykh/mini-humidifier](https://github.com/artem-sedykh/mini-humidifier) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
