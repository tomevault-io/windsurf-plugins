---
trigger: always_on
description: Instructions for AI coding agents (Jules, Codex, Claude Code, Cursor, etc.) working in
---

# AGENTS.md

Instructions for AI coding agents (Jules, Codex, Claude Code, Cursor, etc.) working in
this repo. The goal is for agent-authored changes to be indistinguishable in style and
scope from human-authored ones, so reviewers don't have to launder them through a
second pass.

If a request conflicts with anything below, **stop and ask** rather than guess.

---

## What this project is

SKapsule is an unofficial Android (arm64) port of *Spiral Knights*. The Android app
(`launcher/`) boots a bundled JRE that runs the real desktop game client, with native
shims (gl4es, openal-soft, LWJGL Android, caciocavallo, frenchpress) supplying what
the desktop client expects from its host platform. See [README.md](README.md) for the
full picture and [the layout section](README.md#repository-layout) for what lives
where.

The buildable Android project is `launcher/`. Everything else under the repo root is
either a native-component submodule or a build/CI script.

---

## Build & verification

Don't blind-edit. Before claiming a change works, build the affected component:

- **Launcher / Kotlin / Java (`launcher/`):** `cd launcher && ./gradlew :app:assembleDebug`
  (or `assembleRelease`). Use JDK 25 as `JAVA_HOME`. A successful compile is the
  minimum bar; for behavior changes, also run the relevant unit tests
  (`./gradlew :app:testDebugUnitTest`).
- **Native submodules (`gl4es/`, `openal-soft/`, `lwjgl3/`, `caciocavallo17/`,
  `frenchpress/`):** rebuild via the matching script in `scripts/`. Each script
  expects specific JDKs in its environment — see
  [README §Building from source](README.md#building-from-source-developers) for the
  full matrix. In particular, `build-lwjgl3-android.sh` needs both `JAVA_HOME` →
  JDK 25 and `JAVA8_HOME` → JDK 8 (the JDK-8 `rt.jar` feeds LWJGL's Java-8
  multi-release layer, and no later JDK ships it). If you touch a submodule, the
  *outer* repo must also be updated (the submodule pointer is what CI builds from).
- **CI is the canonical recipe.** [.github/workflows/build-apk.yml](.github/workflows/build-apk.yml)
  is the source of truth for the from-source build. If your change requires a build
  step that isn't in CI, that's a red flag — surface it.

If you cannot run the build in your sandbox, say so explicitly in the PR. Do **not**
claim a change is verified when it isn't.

---

## Scope discipline

The single most common failure mode for agentic PRs in this repo is **overreach**:
splitting one logical change across multiple "micro" PRs, or piling unrelated cleanups
into a feature PR. Don't do either.

- **One PR = one logical change.** A perf pass on a single file is one PR, not three.
  Four trivial commits that each touch `TouchControlOverlay.kt` should have been one.
- **Don't bundle drive-by cleanups** (unused imports, formatting, reorders) into a
  feature/fix PR. If you spot them, mention them in the PR description and let the
  human decide — or open a separate, clearly-scoped `chore:` PR.
- **Don't refactor opportunistically.** If the task is "fix bug X", fix bug X. Don't
  also rename variables, extract helpers, or reflow code that isn't part of the fix.
- **Don't add features, abstractions, or config knobs that weren't asked for.** Three
  similar lines is better than a premature abstraction. No "future-proofing."
- **Don't add error handling, fallbacks, or validation for scenarios that can't
  happen.** Trust internal callers and framework guarantees. Validate only at system
  boundaries.
- **Don't write new files when an existing one will do.** Never create
  documentation/README files unless explicitly asked.

When in doubt about scope, smaller is better, and "ask first" is always allowed.

---

## Commit & PR conventions

Match the existing history. Run `git log --oneline -30` before writing your first
commit message.

**Format:** [Conventional Commits](https://www.conventionalcommits.org/), lowercase
type, lowercase subject, no trailing period, no emoji:

```
feat: add initial support for touch controls
fix: make touch controls work without physical gamepad attached
perf: optimize hot loop in TouchControlOverlay onTouchEvent
chore: bump lwjgl3
test: add unit tests for NativeBridgePrompt fallback logic
```

Common types in this repo: `feat`, `fix`, `perf`, `chore`, `test`, `docs`.

**Subject style:**
- Imperative mood ("add", "fix", "bump") — not "added", "fixes", "bumping".
- Lowercase. No emoji (⚡, 🚀, ✨ etc. — none). No Title Case.
- Aim for ≤ ~72 chars. Backticks around symbol names are fine when they help
  (`` perf: optimize hot loop in TouchControlOverlay `onTouchEvent` ``).

**Bodies:**
- Most commits in this repo are title-only. Don't pad with a body unless there's
  genuine context (the *why*, a non-obvious tradeoff, a linked issue, a workaround for
  an upstream bug). A body that just restates the diff in prose is noise — delete it.

**PR titles:** same rules as commit subjects. **PR descriptions:** explain *why*,
list the user-visible effect, and call out anything reviewers should double-check
(performance assumptions, behavior changes, files you didn't touch but considered).

---

## Code style

### Kotlin / Java (launcher)

- Match the surrounding file. Indentation, brace placement, import order, and naming

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SKonstruct/SKapsule](https://github.com/SKonstruct/SKapsule) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
