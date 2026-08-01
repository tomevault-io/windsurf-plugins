---
trigger: always_on
description: Flutter package providing chat-bubble widgets (WhatsApp-style and other shapes), a message bar, typing indicators, reactions, swipe actions, and message grouping helpers. Published on pub.dev as [`chat_bubbles`](https://pub.dev/packages/chat_bubbles).
---

# chat_bubbles

Flutter package providing chat-bubble widgets (WhatsApp-style and other shapes), a message bar, typing indicators, reactions, swipe actions, and message grouping helpers. Published on pub.dev as [`chat_bubbles`](https://pub.dev/packages/chat_bubbles).

> **Note for AI agents:** this file is the source of truth. `AGENTS.md` is a symlink to it for cross-tool compatibility (Cursor, Aider, Codex, Continue, Copilot Workspace).

## Common commands

Always run from the **repo root**, never from `example/`.

| What | Command |
|---|---|
| Analyze package (must be 0 issues before publish) | `dart analyze lib/` |
| Run widget tests | `flutter test` |
| Auto-fix lints (super-params, etc.) | `dart fix --apply lib/` |
| Resolve deps after pubspec edits | `flutter pub get` |
| Verify publish archive (~3 MB, 0 warnings) | `flutter pub publish --dry-run` |
| Cut a release end-to-end | `./scripts/release.sh <X.Y.Z>` |
| Run example app on web | `cd example && flutter run -d chrome` |

## Agent ground rules

- **Never `git push` to `master` directly.** Master is updated via PR from `develop` only.
- **Never run `flutter pub publish` without explicit user approval.** A release is a one-way action.
- **Never bump the major version without asking.** Major bumps drop users on older SDKs.
- **Never amend or force-push commits on a shared branch.** Local-only commits may be amended freely.
- **Always run `dart analyze lib/` before claiming a change is done.** A "0 issues" result is the bar for any code edit.
- **When adding a public class to `lib/`, you must also add an `export` line** to `lib/chat_bubbles.dart`. Forgetting this means consumers can't reach the class — caught at v1.10.0.
- **Use `Color.withValues(alpha: x)`, never `Color.withOpacity(x)`.** The latter is deprecated and tanks the pana score.
- **Date format in `CHANGELOG.md` is `DD/MM/YYYY`** (e.g. `01/06/2026`). Not ISO.

## Tech & constraints

- **SDK:** Dart `>=3.6.0 <4.0.0`, Flutter `>=3.27.0` (required for `Color.withValues()`).
- **Runtime dependency:** `intl: ^0.20.1` only. The package deliberately keeps zero heavy deps — anything audio/image-related is the consumer's responsibility.
- **Lints:** `flutter_lints` + `public_member_api_docs`. `dart analyze lib/` must report **zero issues** before publishing.

## Repository layout

```
.
├── lib/                          # Published source
│   ├── chat_bubbles.dart         # Public barrel — every consumer-facing widget is re-exported here
│   ├── algo/                     # Date-chip text helpers
│   ├── bubbles/                  # All bubble shapes (normal, special_{one,two,three}, audio, image, reply, link_preview)
│   ├── date_chips/               # DateChip widget
│   ├── groups/                   # BubbleGroupBuilder + MessageGroupHelper (consecutive-sender clustering)
│   ├── indicators/               # TypingIndicator + TypingIndicatorWave
│   ├── message_bars/             # MessageBar + MessageBarStyle
│   ├── reactions/                # BubbleReaction, Reaction, ReactionPicker, ReactionOverlay
│   ├── swipe/                    # SwipeableBubble wrapper for reply/delete gestures
│   └── utils/                    # Internal helpers (status row, forwarded header) — NOT exported
├── example/                      # Runnable Flutter app showing every widget. Source-only ships to pub
├── test/                         # Widget tests
├── images/                       # README assets + pub.dev screenshot (images/logo/logo.png)
├── pubspec.yaml                  # Package manifest
├── analysis_options.yaml         # Lint config
├── .pubignore                    # Critical — excludes build/, .dart_tool/, example platform folders
├── CHANGELOG.md                  # Required by pub.dev; one section per version (newest first)
└── README.md                     # Renders on pub.dev landing page
```

### What gets published

The `.pubignore` (NOT `.gitignore`) controls the published archive. It excludes:
- `**/build/`, `**/.dart_tool/`, `**/pubspec.lock`, IDE files, `.DS_Store`
- `example/android/`, `example/ios/`, `example/web/`, etc. — only `example/lib/`, `example/pubspec.yaml`, `example/README.md` ship

A healthy archive is **~3 MB**. If `flutter pub publish --dry-run` reports significantly more, something is leaking in — fix `.pubignore` before publishing. Background: v1.9.0 shipped ~50 MB of leaked build artifacts and pana timed out, breaking the score page. The `.pubignore` was added in v1.9.1 to fix this.

## Branching & release flow

- `master` — published / stable. Tag releases here (`v1.10.0` etc.).
- `develop` — integration branch. All feature PRs target `develop`.
- Contributor PRs go to `master` directly when they're external (e.g. #64).
- Release flow: bump version on `develop` → push → merge `develop` → `master` via PR → tag → publish.

Commit-message convention: short subject line. Release commits are titled simply `v<version>` (e.g. `v1.10.0`).

## Publishing checklist

Run from the **repo root** (not from `example/` — that publishes the example app and fails with confusing errors):

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [prahack/chat_bubbles](https://github.com/prahack/chat_bubbles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
