---
trigger: always_on
description: Reference for AI coding agents working on this codebase. Keep this file concise, prescriptive, and focused on repository-wide rules.
---

# Mercury — Agent Engineering Notes

Reference for AI coding agents working on this codebase. Keep this file concise, prescriptive, and focused on repository-wide rules.

---

## 1. Communication and Documentation

- Communicate with the user in Chinese.
- Write code comments and repository documentation in English unless explicitly requested otherwise.
- Do not use emojis in code comments or documentation.
- Use backticks for code references in Markdown.

---

## 2. Core Stack and Naming

- Platform: macOS-first, no iOS target.
- Language: Swift.
- UI: `SwiftUI` first. Use `UIKit` / `AppKit` only when unavoidable.
- Networking: `URLSession`.
- Storage: `SQLite` + `GRDB`. `CoreData` is fallback only.
- Feed parsing: `FeedKit`.
- HTML cleaning: `SwiftSoup`.
- Article extraction: in-house `Readability`, no `WebKit` dependency.
- Markdown to HTML: `swift-markdown` + internal `MarkupHTMLVisitor` (GFM-aware). Rendered HTML is cached by `themeId + entryId` with `readerRenderVersion`; see `docs/markdown-engine.md`.
- LLM client: `SwiftOpenAI`.
- Default numeric type is `Double`; use `CGFloat` only when required by API.

Naming and file structure:

- Keep source layout flat unless a deeper module is clearly reusable.
- SwiftUI view files use `*View.swift`; view models use `*ViewModel.swift`.
- Shared agent infrastructure uses `Agent*` prefix.
- Feature-specific files use feature prefixes such as `Summary*` and `Translation*`.
- `AppModel` extensions use `AppModel+FeatureName.swift`.
- `AI*` prefixes are deprecated; use `Agent*`.

`SwiftOpenAI` routing note: request building replaces base URL path. Preserve provider paths via `overrideBaseURL + proxyPath` plus version segment when needed, otherwise compatible providers may return `404`.

---

## 3. Build and Verification

Run from repo root:

```shell
./scripts/build
```

- Do not pipe or post-process `./scripts/build` output.
- Keep the build free of compiler errors and warnings.
- If tooling returns empty or missing output, stop and ask the user to verify manually.

---

## 4. Localization Rules

Full design: `docs/l10n.md`.

- All user-visible strings must resolve through `LanguageManager.shared.bundle`.
  - Use `Text(..., bundle:)` in views.
  - Use `String(localized:..., bundle:)` or equivalent bundle-aware APIs in model/runtime code.
- Never localize debug issue strings.
- Avoid runtime-computed `LocalizedStringKey`; prefer static keys or bundle-resolved `String` values.
- `View.help()`, some `Picker` convenience initializers, and `.tabItem` ignore the environment bundle; pass pre-resolved `String` values there.
- Prefer extraction-friendly localization call sites: avoid hiding keys inside ternaries or other dynamic expressions when a static helper/property can expose each key literally.
- Do not manually edit `Localizable.xcstrings` extraction metadata such as `extractionState` / `stale`; let Xcode extraction and build tooling maintain those fields.
- Keep SwiftUI imports out of pure model files; move UI-facing label keys or presentation helpers into view-facing extensions when needed.

Must-use shared facilities:

- Reuse existing shared localization/presentation helpers when the semantics already exist.
- Do not introduce feature-local copies of prompt fallback, failure, status, or task-title wording when a shared projection/helper already covers that concept.

---

## 5. Agent Task Shared Facilities

Agent-task behavior must converge on shared infrastructure instead of re-implementing feature-local variants.

Mandatory shared facilities:

- `TaskQueue` / `TaskCenter` for background and long-running orchestration.
- `AgentRunStateMachine` for state transitions.
- `AgentRuntimeEngine` for active/waiting lifecycle management.
- `AgentRuntimeStore` for in-memory runtime indexing.
- `AgentExecutionShared` for shared route resolution and terminal recording.
- `AppTaskKind.displayTitle` for task display titles.
- `AgentRuntimeProjection` for shared user-facing failure and runtime projection logic.
- `AgentPromptCustomizationConfig` and related prompt-template facilities for prompt fallback behavior.

Hard rules:

- Do not build ad-hoc task schedulers or parallel lifecycle systems outside the shared task/runtime stack.
- Do not create task-local copies of failure-message projection when `AgentRuntimeProjection` already owns that semantic.
- Do not create task-local copies of prompt fallback wording when shared prompt-customization helpers already exist.
- Do not hard-code new task titles inside executors when `AppTaskKind.displayTitle` is available.
- When adding a new reusable agent-task semantic, extend the shared facility first instead of forking it locally in one feature.
- Do not route new app-authored string literals through a freeform message escape hatch; migration-only usage must be removed within the same unification iteration, and any truly non-structurable case requires explicit discussion.

---

## 6. Agent Runtime Contracts

Core architecture:

- `AgentRunStateMachine`: pure transitions.
- `AgentRuntimeEngine`: lifecycle driver.
- `AgentRuntimeStore`: active/waiting index.
- `AgentExecutionShared`: shared route resolution and terminal recording.

Global execution policy:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/neolee) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
