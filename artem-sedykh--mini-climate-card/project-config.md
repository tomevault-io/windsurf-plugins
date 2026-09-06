---
trigger: always_on
description: Guidance for AI coding agents working on this repository. Human contributors
---

# AGENTS.md

Guidance for AI coding agents working on this repository. Human contributors
are welcome to read it too - it is the short version of how this card is put
together and what breaks it.

## What this is

`mini-climate-card` is a custom Lovelace card for Home Assistant: a single
JavaScript bundle that Home Assistant loads in the browser. There is no server
side, no Python, and no Home Assistant integration in this repository. The card
reads entity state out of the `hass` object and calls Home Assistant services
back.

Distribution is HACS: users get `mini-climate-card-bundle.js` as a release
asset, so **the release asset is the product**. A change is not shipped until
it is in a tagged release.

## The card is configured, not coded

What this card is, is a kit. An `entity:` and nothing else gives a working
card, but almost everything on it can be described in the user's YAML instead:
which indicators exist and what each one reads, which buttons appear, what icon
each takes, what service each one calls, when each is hidden or disabled.

Several of those options are **template strings**. The user writes a function
as text in YAML, and `compileTemplate` in `src/utils/utils.ts` re-parses it
with `new Function` and calls it with a context object. That is the one
deliberate `new Function` in the repository.

This is a constraint on what may be changed, not just a description of what is:

- **The configuration is open at the leaves.** Indicator and button ids are
  chosen by the user, and their options are templates the user wrote. Anything
  that validates a configuration, or edits one, has to carry through what it
  does not recognise rather than dropping it. An editor that round-trips a
  hand-written card through a form and silently loses the half it does not
  model is worse than no editor.
- **A template is source text, not a function.** It is compiled at
  `setConfig` time, before the card has a `hass`, which is why the context
  passes `call_service` as a closure rather than a value.

#### Better a short answer than a wrong link

When pointing a reader at documentation, verify the anchor is there before
writing it. Two of these replies in one day pointed at a section that did not
yet exist and at an option that was not in the docs - and a dead link is worse
than no link, because it sends a person to a page that does not have what was
promised. The rule here is the same as for the answers themselves: an answer is
not real until a test holds it up. A documentation link is not real until a
check says it exists - and `check:docs` cannot be the one to say it, because it
does not know prose.

## The template context

Every compiled template is called with `this` bound to a context built in
`src/main.ts`, and the context is `{ ...value }` - **the option's own YAML
spread into it**. So anything the user writes beside a template is readable
from the template as `this.<key>`. That is not an accident to tidy up; it is
the extension point, and it is why unknown keys inside an indicator or a button
cannot be rejected.

On top of that the context carries:

- `entity_config` - the whole card configuration.
- `toggle_state` - the `on`/`off` flip from `src/utils/utils.ts`.
- `call_service(domain, service, options)` - for buttons and target
  temperature only. Indicators do not get it, because an indicator displays.

## Language

**English only**, everywhere: code, comments, commit messages, issues, pull
requests, documentation, and release notes. This is a public repository with
external contributors who do not read Russian.

## Commands

```
npm ci                # install exactly what the lockfile says
npm run lint          # eslint 10, flat config
npm run format        # prettier --write
npm run format:check  # what CI runs
npm run typecheck     # tsc --noEmit; esbuild strips types, it never checks
npm test              # vitest, the unit tests under test/
npm run test:coverage # the same, with coverage and its thresholds
npm run test:watch    # vitest in watch mode
npm run test:browser  # @web/test-runner, the component tests in two engines
npm run bench up      # a Home Assistant of its own in docker; down to remove it
npm run test:e2e      # the scenarios against that instance (needs a bench up)
npm run rollup        # bundle src/main.ts -> dist/mini-climate-card-bundle.js
npm run dev           # the same bundle, unminified
npm run check:bundle  # assertions on the built bundle (needs a build first)
npm run check:options # every option the card reads is documented, and vice versa
npm run build         # lint + format:check + test + rollup + check:bundle
npm run watch         # unminified, rebuilding on save
```

Node version comes from `.nvmrc`. Use it; CI reads the same file.

There are **four layers of checks** - see "Checks" below. The fourth, the
bench, is the only one that renders the card inside a real Home Assistant. It
needs docker, and CI runs it in a workflow of its own rather than in the
required check.

## Checks

Four layers, in the order of how much they cost to run: assertions on the built
bundle, unit tests, the card rendered in two browser engines, and the card on a
dashboard in a Home Assistant of its own. CI runs all four - the first three in

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [artem-sedykh/mini-climate-card](https://github.com/artem-sedykh/mini-climate-card) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
