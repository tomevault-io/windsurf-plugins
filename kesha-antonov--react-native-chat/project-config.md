---
trigger: always_on
description: Guidance for AI coding agents working **inside this repository**.
---

# AGENTS.md

Guidance for AI coding agents working **inside this repository**.

If you are instead helping someone *use* this library in their own app, read
[`llms.txt`](llms.txt) - it is a compact integration guide and does not require the checkout.

## What this is

`@kesha-antonov/react-native-chat` is a chat UI component library for React Native and
Web - a maintained continuation of `react-native-gifted-chat`. It ships as compiled
JavaScript plus type definitions; there is no native code in this package.

## Setup

Requires **Node >= 20** and **Yarn 4** (pinned via `packageManager`; use `yarn`, never `npm`).

```bash
yarn install          # library
cd example && yarn install && cd ..   # example app - a SEPARATE yarn project
```

The repo is **not** a Yarn workspace. `example/` has its own `package.json`, its own
`yarn.lock`, and its own `node_modules`, and depends on the library via `link:..`. Installing
at the root does not install the example, and vice versa.

## Commands

Run these from the repo root:

| Command | What it does |
| --- | --- |
| `yarn test` | Jest suite, run under `TZ=Europe/Paris` (snapshots contain formatted times, so the timezone is pinned). Must stay green. |
| `yarn test:watch` / `yarn test:coverage` | Same suite, watching or with coverage. |
| `yarn typecheck` | `tsc --noEmit` over `src/`. Must stay clean. |
| `yarn lint` | ESLint over **both** `src/` and `example/`. Must report 0 errors and 0 warnings. |
| `yarn lint:fix` | Auto-fix what ESLint can. |
| `yarn build` | `rm -rf lib && tsc` - emits `lib/`, which is gitignored and is what gets published. |
| `yarn prepublishOnly` | lint + test + build, i.e. the full gate. |

Inside `example/`: `yarn lint` and `yarn typecheck` cover the example app on its own. Note
that the root `yarn lint` also lints `example/`, under **stricter** rules than the example's
own `expo lint` - so a change that passes `cd example && yarn lint` can still fail at the root.
Always run the root `yarn lint` before you finish.

A husky `pre-commit` hook runs `lint-staged`. Its glob is `src/*.{json,js,jsx,ts,tsx}`, which
matches only the **top level** of `src/` - a change under `src/MessagesContainer/` or
`src/components/` is committed without `lint:fix` touching it. Do not rely on the hook; run
the root `yarn lint` yourself.

## Layout

```
src/                     library source - the only thing published (as compiled lib/)
  Chat/                  the top-level <Chat> component
  MessagesContainer/     list engine (FlatList / FlashList), day header, scroll handling
  Bubble/ Message/       a single message row and its bubble
  Day/                   the date separator; the animated floating header lives in
                         MessagesContainer/components/DayAnimated
  Reactions/ Reply/      emoji reactions, swipe-to-reply
  TypingIndicator/       the three-dot bubble
  components/            shared leaf components (Icon, TouchableOpacity, markdown, voice…)
  hooks/                 useTheme, useLabels, useStreamingMessages, …
  locales/ i18n.ts       15 built-in UI translations, one file per language; en is the default
  Theme.ts Icons.ts      theme tokens and the overridable icon registry
  rtl.ts                 RTL detection and position mirroring
  linkParser.tsx         URL / phone / email / mention / hashtag matching for MessageText
  logging.ts             branded warning() / error() helpers
  Models.ts              IMessage, User, QuickReplies, MessageReaction - the public data model
  index.ts types.ts      public API surface; anything not exported here is internal
  __tests__/             Jest tests, colocated with the source they cover
tests/setup.ts           global Jest mocks (reanimated, worklets, safe-area, keyboard)
example/                 Expo demo app - separate yarn project, consumes the built lib/
expoSnack/               single-file demo for snack.expo.dev; not linted or built by CI
docs/                    MIGRATION.md, STREAMING.md
```

A day separator is rendered by the list itself (`MessagesContainer/components/Item`), guarded
by `isSameDay`, *around* whatever `renderMessage` returns. A custom `renderMessage` must not
render its own `<Day>` - it would print a pill above every message.

## Conventions

Style is enforced by ESLint (`@stylistic`), so run `yarn lint:fix` rather than matching by eye:

- **no semicolons**, single quotes, 2-space indent, single quotes in JSX
- a space before a function's parameter list: `export function useThemeColor (props) {`
- **no braces around a single-statement block**, and the statement goes on the next line
  (`curly: multi` + `nonblock-statement-body-position: below`):
  ```ts
  if (!currentMessage?.createdAt || isSameDay(currentMessage, previousMessage))
    return null
  ```
- trailing commas on multiline arrays/objects/imports, none on function args; arrow params
  are unparenthesised when there is exactly one (`arrow-parens: as-needed`)
- imports are sorted by `perfectionist/sort-imports` - `yarn lint:fix` will reorder them
- comments explain *why*, not *what*; several existing ones cite the issue they fix
- user-visible diagnostics go through `logging.ts` (`warning`, `error`), which prefixes them
  with the package name, rather than a bare `console.*`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kesha-antonov/react-native-chat](https://github.com/kesha-antonov/react-native-chat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
