---
trigger: always_on
description: Read this before adding or changing any user-facing text in `apps/ui`.
---


# Translations (Lingui + Weblate)

Read this before adding or changing any user-facing text in `apps/ui`.
Catalogs live in `apps/ui/src/locales/*.po` and are translated on
[Weblate](https://hosted.weblate.org/engage/maintainerr/).

---

## A translation must never change behavior

Translate text the user reads, and nothing else. A string that is displayed
*and* used for something else stops being display text: switching language then
changes what the app does, and no test written in English can see it.

Before wrapping a string, follow where the value goes. It must not reach:

- **Anything persisted or sent to a server.** A default collection name built
  from a translated word is stored on Plex in whatever language the form was
  submitted in.
- **Anything parsed or compared.** `startsWith`, `includes`, `===`, a `switch`,
  `.toLowerCase()` - a prefix agreed between two functions must be a constant
  they share, never a message.
- **Any key or identifier.** Grouping keys, `Map`/`Set` members, TanStack Query
  keys, React keys, DOM `id`/`name`. Key on the enum or id the label was
  derived from, and translate only at the point of display.
- **Any hook dependency array that guards work.** See below.

Sorting a list by its displayed labels is fine - that is display order, and it
is *meant* to follow the language.

## Where a translation resolves

`useLingui()`'s `t` is a render-scoped value: it changes when the locale
changes, which is exactly why a component re-renders in the new language. That
also makes it a dependency.

- **In render** - use the hook's `t`. The component re-renders on a language
  switch and the text follows.
- **In an effect, an async callback, or an event handler** - use `t` from
  `@lingui/core/macro`, aliased as `globalT` when the file also uses the hook.
  It resolves against the active locale when it runs, and it is not reactive,
  so it stays out of the dependency array.

When aliasing an existing file to `globalT`, rename the call sites by hand.
A blind replace of ``t` `` also rewrites the `t` that ends a message
(``t`Delete Soonest` `` becomes ``globalT`Delete SoonesglobalT` ``), which still
compiles and still renders text. Diff the message strings against the previous
revision afterwards.

Putting the hook's `t` in an effect's dependencies makes that effect re-run on
every language switch: re-fetching data, re-firing toasts, and in one case
reloading a form and discarding the user's unsaved edits.

A component that renders translated text must consume the i18n context, or a
language switch will not reach it. `I18nProvider` re-renders context consumers
only - React reuses the untouched children subtree - so a component that calls
core-macro `t` in its render path and never calls `useLingui()` keeps the
language it first mounted with. Navigation hides this by remounting the page;
anything mounted during the switch shows it.

## Which macro

| Where | Import | Use |
| --- | --- | --- |
| Inside a React component | `@lingui/react/macro` | `<Trans>` for JSX text, `` t` ` `` from `useLingui()` for attributes, toasts, handlers, `<Plural>` for counts |
| Plain module (`utils/`, `api/`, helpers) | `@lingui/core/macro` | `` t` ` `` / `plural()` - resolves when the function runs |
| A value held before render | `@lingui/core/macro` | `msg` descriptor, translated at the call site with `t(descriptor)` |

Never call the runtime `i18n._()` directly - `useLingui()`'s `t` resolves a
descriptor just as well and keeps the underscore API out of the codebase.

## Module scope freezes a translation

A `const` holding `` t` ` `` is evaluated once, at import, in whichever locale
loaded first. Anything defined outside a render must be either a `msg`
descriptor or a builder function called during render:

```ts
// wrong - frozen at import
const OPTIONS = [{ value: 'movie', label: t`Movies` }]

// right - resolved per render
const buildOptions = () => [{ value: 'movie', label: t`Movies` }]
```

The same applies to **default parameter values**, which run before
`useLingui()`. Resolve them in the body instead: `const label = props.label ?? t\`Save\``.

## Placeholders

- Give every placeholder a name. Hoist member expressions into a local, or use
  the labelled form: `` t`Failed to reach ${{ serverName }}` ``. An unnamed
  `{0}` tells a translator nothing. Keep the **whole** expression when you
  hoist - `props.collection.title`, never `props.collection`.
- **A single quote is an ICU escape character.** `'{name}'` renders the literal
  text `{name}` and eats the quotes, silently dropping the value. Write a
  literal apostrophe beside a placeholder as `''` (`&apos;&apos;` in JSX). A
  lone apostrophe not touching a brace ("don't") is fine.
- `<Plural one="... {name}">` does **not** bind `name`. Use `plural()` from the
  core macro with the placeholder inside each choice instead.

## Counted text

Never build a plural by appending `s`. `${n} item${n === 1 ? '' : 's'}` is
wrong in most languages - use `plural()` or `<Plural>` so translators can
supply their own forms.

## Whole sentences, not fragments

A sentence split across several messages cannot be reordered by a translator.
Keep inline markup and values inside one `<Trans>`. Where the original built a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Maintainerr/Maintainerr](https://github.com/Maintainerr/Maintainerr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
