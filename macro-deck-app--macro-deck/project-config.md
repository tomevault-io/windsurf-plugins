---
trigger: always_on
description: Macro Deck 3: .NET host (owns state and business logic), an Angular desktop UI and a framework-free
---

# Claude Code guidance

Macro Deck 3: .NET host (owns state and business logic), an Angular desktop UI and a framework-free
web client (REST + JSON WebSocket), Tauri bootstrapper (owns the native window and host process).

## Workflow

Branch before working. Prefix `feature/`, `fix/`, `refactor/`, `chore/`, `docs/`, `ci/` plus a short
kebab-case description and the issue number when one exists.

## Public SDK and plugin compatibility

A plugin may stay compiled against an old SDK indefinitely. Every non-obsolete public plugin-facing
contract is a compatibility commitment: `sdk/`, `ui-model/`, `protocol/`, plugin HTTP/WebSocket
behaviour, manifest and package formats, analyzer diagnostic ids, conformance check ids. Source and
binary compatibility both count.

**If a fix needs to break one, stop and ask before implementing it.** Protocol breaks require a new
protocol major with the previous version still served.

When these surfaces change, run the compatibility, conformance, deprecation-lifecycle, protocol, and
package tests, and add a regression test from the old consumer's point of view.

See [`docs/`](docs/) and [ADR 0026](engineering/decisions/0026-plugin-protocol-and-sdk-boundary.md).

## Testing

Derive expectations from the requirement or contract, never from the current implementation. When a
test and the implementation disagree, go back to the requirement instead of editing either side.

Prefer a few meaningful scenarios to coverage. Do not test private methods, internal call sequences,
trivial accessors, or framework behaviour; mock real external boundaries only.

## Localization

**No user-facing hardcoded strings.** Anything a person can read in the app (labels, headings, buttons,
tooltips, aria-labels, placeholders, dialog and toast text, validation and error messages, empty states,
status text, integration action and parameter names, native menu and dialog text) comes from a
localization resource, and ships with a translation in **every** language the app already carries. English
is the default and the fallback; a new key missing a value in any other shipped language (German, Italian,
Czech, Polish, Spanish, French) is an incomplete change, not a follow-up.

Internal strings are exempt and stay as they are: `ILogger` messages, exception text that is only ever
diagnostic, protocol and enum values, error *codes*, identifiers, config keys, routes, storage keys, CSS
class names, icon names, and test ids.

New keys go in `macrodeck.app` (`host/src/MacroDeckHost.Localization/Localization/`), reached through the
generated `AppStrings` in C#, `ClientAppStrings` from `@macro-deck/runtime` in the web client, and
`AppStrings` / `'macrodeck.app:Key' | translate` in Angular. The web client compiles in only the slice of
the catalog it can paint - the whole thing is half a megabyte of strings a deck never shows - so a key it
needs must sit under one of the namespaces listed in `GeneratedTypeScriptDriftTests`, or that list grows
and the modules are regenerated. Only add to
`macrodeck` (`sdk/src/MacroDeck.Localization/Resources/`) when the string is genuinely reusable by
plugins: that catalog is published in the SDK, and its keys are a frozen, additive-only contract.

Reuse an existing key rather than minting a duplicate, keep one key per sentence with `{placeholders}`
for values instead of concatenating translated fragments, and use a `[plural]` family for anything that
counts.

Translate for meaning, not word by word: a translated label may be phrased differently from the English
one where that reads better. Each shipped language has its own established register, kept consistent
across the whole catalog:

- German, Italian, Spanish, French: informal address (`du`/`tu`/`tú`/`tu`), imperatives without a pronoun
  (`Wähle ein Ereignis`, not `Wählen Sie ein Ereignis`), lowercase mid-sentence for German.
- Czech, Polish: impersonal phrasing, meaning imperative verb forms for actions and impersonal statuses/errors
  rather than direct `ty` address, matching each language's own desktop-software convention.

Czech and Polish also need a plural-form adjustment the other languages don't: the localization compiler
only distinguishes `count == 1` from every other count (see
[the localization guide](docs/src/content/docs/features/localization.md#only-one-and-other)), which is
grammatically exact for German/Italian/Spanish/French but not for Czech/Polish's `few`/`many` forms. Phrase
a Czech or Polish `Other` form to avoid noun-count agreement (a count-agnostic label rather than a declined
noun) so it stays grammatical for every count.

After changing a resource, regenerate the checked-in TypeScript and Rust catalogs:

```bash
MACRODECK_UPDATE_GENERATED=1 dotnet test sdk/tests/MacroDeck.Localization.Tests.UnitTests
```

See [`docs/src/content/docs/features/localization.md`](docs/src/content/docs/features/localization.md).

## Icons

App icons come from [Lucide](https://lucide.dev), which includes the Feather set. When Lucide has no icon
for the concept, draw one in the same style: 24x24 viewBox, no fill, stroke width 2, round caps and joins.
Mask icons in `ui/runtime/styles/icons/` use `stroke="#000"`; icons shown as images set the colour they need.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Macro-Deck-App/Macro-Deck](https://github.com/Macro-Deck-App/Macro-Deck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
