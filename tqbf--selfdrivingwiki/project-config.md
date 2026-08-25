---
trigger: always_on
description: Last verified: 2026-08-13
---

Last verified: 2026-08-13

* Running `/graphify` is always permitted, including from read-only branches or other read-only repository states, when `tmp/graphify-out` is specified as the output directory.

* **All Swift code must compile via SwiftPM from the command line — prefer
  `make build` / `make test` as the default entrypoints, and ensure bare
  `swift build` / `swift test` also work when used directly. Never rely on
  Xcode-only tooling or APIs.**
  In practice this means: no macros or APIs that require an Xcode project
  file, an Xcode-managed scheme, or Xcode's build system to resolve (e.g.
  avoid `@Entry` for `EnvironmentValues`/`FocusedValues` — use the manual
  `EnvironmentKey`/`FocusedValueKey` pattern instead). If a feature only
  builds inside Xcode, it doesn't belong in this codebase.

* For all coding tasks use your judgement to decide an appropriate lower
  power model and run that in a subagent.

* Keep a master PLAN.md as an index to the documentation you make for this codebase.

* Store specific in-depth documentation in plans/whatever.md.

* **Supported platform and required gates:** Self Driving Wiki supports macOS
  only. Required SwiftPM build/test gates and the required `swift` workflow job
  run on macOS; Linux portability is an optional Apple Container/Docker
  diagnostic and is not a product gate.

* Record progress in `progress/`. I should be able to tell a future agent
  "read PLAN.md and progress/" and trust it is up to speed with this codebase.

* `PLAN.md` and `PROGRESS.md` document features and design-relevant refactorings only.
  Do not add bug-fix entries to either file.

* When writing prose, use the ste-writing skill to ensure clarity.  This applies to pull requests, 
  github issues, plans in the plans folder, and progress in PROGRESS.md.

* Before and after deciding on code, use the swiftui-pro skill to ensure we're following
  modern best practices.

* When setting type, use the typography-designer skill to make sure we're using consistent
  type scales with a sensible visual hierarchy. Pay attention to type weight and emphasis.

* Use the macos-design skill to make sure the UI we come up with makes sense as a modern
  macOS app, with modern professional macOS idioms. Keep things simple.

## Modeling rules

* **Page and source IDs are separate namespaces.** Use `PageID` for rows in
  `pages` and `SourceID` for rows in `sources`; construct either from raw text
  only at persistence or external-format boundaries. Source content-version
  IDs, source Markdown-version IDs, and chat IDs remain `PageID` until their
  own namespace migrations. Mixed targets use tagged enums such as
  `BookmarkNode.Content`. The raw ULID strings and existing JSON, SQLite,
  wiki-link, File Provider, CLI, and staging formats are compatibility
  contracts; see `plans/page-source-id-separation.md`.

* **Avoid stringly-typed variables.** A bare `String` (or `String?`) that
  actually means "a chat id" / "a queue item id" / "a provider name" makes two
  different id spaces compare equal, lets a typo pass the type checker, and
  gives the reader no idea what values are legal. Wrap it: a `RawRepresentable`
  struct (`PageID`) when it's one id space, an `enum` when it's a closed set of
  cases, and a **namespaced enum** when one field must carry ids from several
  spaces (`TranscriptID.chat(PageID)` / `.queueItem(QueueItem.ID)` — the case
  tag is what makes a chat ULID unable to collide with a queue ULID). Same for
  file paths (`URL`), durations (`Duration`), and raw enum-backed strings —
  convert at the boundary, not at every use site.

  **A sentinel string is the same smell.** `ChatSessionKey.draft` /
  `.chat(PageID)` replaced a `String` key whose draft was spelled
  `"__wiki_draft_chat__"`: that sentinel shared a namespace with real chat
  ULIDs, so "is this the draft?" was a comparison against a magic constant that
  every call site had to remember. Give the special case its own case, and the
  compiler asks the question for you — `.draft` carries no `PageID`, so a draft
  cannot satisfy an id comparison even by accident.

* **Prefer enums (or named constants) over magic numbers.** A bare literal at a
  use site — `if attempt > 3`, `rowHeight = 36`, `case 2: …` — hides both its
  meaning and the fact that the same number is load-bearing somewhere else. If
  it names one of a closed set of choices, make it an `enum`; if it's a tuning
  value, hoist it to a named `static let` in the type or metrics enum that owns
  it (`ChatMetrics`, `PageEditorMetrics`) so the name carries the rationale and
  there is exactly one place to change it. The same goes for raw values
  crossing a boundary: decode into an `enum` at the edge rather than comparing
  integers or strings downstream.

* **Prefer a finite state machine over a cluster of flags.** Several
  `Bool`/optional properties that are really one lifecycle are a denormalized
  enum: they can encode impossible combinations, and every write site has to
  remember to update all of them coherently (see `ChatRunState`, which replaced
  `isRunning`/`isGenerating`/`isAwaitingGenerationSlot`/`isInteractiveSession`/
  `activeChatID` after a missed write in one of them shipped a bug). Model the
  states as an `enum`, make it the single stored source of truth, and derive

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tqbf/selfdrivingwiki](https://github.com/tqbf/selfdrivingwiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
