---
trigger: always_on
description: ./gradlew formatKotlin
---

# Copilot Instructions - android-compose-highlight

## Build, test, and lint commands

```bash
# Lint (ktlint via kotlinter) - must pass before commit
./gradlew lintKotlin

# Auto-fix formatting
./gradlew formatKotlin

# Lint markdown (markdownlint-cli) - validates CHANGELOG.md and other markdown files
npx markdownlint-cli CHANGELOG.md

# Auto-fix markdown formatting
npx markdownlint-cli --fix CHANGELOG.md

# JVM unit tests (fast, no device needed)
./gradlew :compose-highlight:test

# Run a single test class
./gradlew :compose-highlight:test --tests "dev.hossain.highlight.engine.ThemeParserTest"

# Run a single test method
./gradlew :compose-highlight:test --tests "dev.hossain.highlight.engine.ThemeParserTest.parse returns non-empty map for valid CSS"

# Build the library AAR
./gradlew :compose-highlight:assembleDebug

# Build the sample app
./gradlew :sample:assembleDebug

# JVM microbenchmark for HtmlToAnnotatedString (no device needed, opt-in via flag).
# Skipped from regular `:test` runs via Assume.assumeTrue, so the default test suite stays fast.
# Writes a JSON report to compose-highlight/build/reports/benchmarks/html-parser-baseline-<epoch-ms>.json
# with mean/stddev/min/max in microseconds for each of the 12 @Test methods (6 single-theme convert
# + 6 dual-theme convertBothThemes against the real-world hljs HTML fixtures).
# Use this to catch parser regressions: capture a baseline, swap the parser, re-run, diff the JSON.
./gradlew :compose-highlight:testDebugUnitTest \
  --tests "dev.hossain.highlight.benchmark.HtmlParserBenchmark" \
  -PrunBenchmark=true --rerun-tasks

# Run microbenchmarks on a connected device (requires physical device or emulator)
./gradlew :compose-highlight:connectedAndroidTest

# Run a specific benchmark class
./gradlew :compose-highlight:connectedAndroidTest \
  -P android.testInstrumentationRunnerArguments.class=dev.hossain.highlight.benchmark.HighlightEngineBenchmark

# Generate Dokka API docs → docs/api/
./gradlew :compose-highlight:dokkaGeneratePublicationHtml
```

## CHANGELOG.md Markdown Formatting

**Markdown linting usage:**
- Run `npx markdownlint CHANGELOG.md` to validate all formatting rules before committing
- The repository includes `.markdownlintrc` configuration that enforces changelog-specific rules

**.markdownlintrc configuration:**
- `MD013`: Line length limit of 120 characters (vs. default 80) to accommodate detailed technical changelog entries
- `MD024`: `siblings_only: true` - Prevents duplicate section headings (e.g., multiple `### Fixed`) within the same release version. Allows the same heading across different versions, which is normal for changelogs

**CHANGELOG.md formatting rules:**
1. **No duplicate headings within a version** - Each release (`## [X.Y.Z]`) should have at most one of each section heading type (`### Fixed`, `### Added`, `### Changed`, etc.). If your changes span multiple categories, consolidate them under a single section heading with multiple bullet items. This rule is enforced by MD024 with `siblings_only: true`.
2. **Blank lines around section headings** - Every heading (`### Fixed`, `### Added`, `### Changed`, `### Performance`, `### Infrastructure`, etc.) should be surrounded by blank lines (one before, one after).
3. **Blank lines around code fences** - All code blocks (triple backticks) should be surrounded by blank lines.
4. **Line length** - Keep lines reasonably wrapped for readability; long bullet items should wrap to multiple indented lines instead of becoming hard to review. The 120-character limit allows technical explanations to stay together without excessive wrapping.
5. **Long bullet wrapping pattern** - Convert long single-line items like:

   ```
   - **Item** - [500+ char explanation with multiple concepts]
   ```

   To indented multi-line format:

   ```
   - **Item** - [Explanation line 1]
     [Explanation line 2 indented by 2 spaces]
     [Explanation line 3 indented by 2 spaces]
   ```

6. **No spaces inside code spans** - Code spans must not have spaces between backticks and content: `` `code` `` not `` ` code ` ``.
7. **Run `markdownlint` before committing** - Always verify no violations remain: `npx markdownlint CHANGELOG.md`

## Docs Changelog Sync

**Keep `docs/changelog.md` in sync with root `CHANGELOG.md`.** The docs site file serves as a curated summary for visitors - it should always contain brief highlights of the **last 5 releases** only. After each release:

1. Add a new `### X.Y.Z - Brief Title` section at the top of the "Recent highlights" block in `docs/changelog.md`
2. Extract 3-5 key bullet points from the root `CHANGELOG.md` for that release (focus on user-facing features and major fixes)
3. Keep descriptions brief (1-2 lines per item, no wrapped multi-line format)
4. Remove the oldest release entry to maintain the "last 5 releases" limit
5. No markdown linting required for `docs/changelog.md` - it's documentation, not a strict changelog

Example format:
```markdown
### 0.25.0 - New theme support

- Added 10 new built-in themes (Dracula, Solarized, etc.)
- Fixed WebView crash on Android 12 devices
- Improved theme parsing performance by 40%
```

## Architecture

The library has two layers — `engine/` (public engine API, with `engine/internal/` for

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hossain-khan/android-compose-highlight](https://github.com/hossain-khan/android-compose-highlight) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
