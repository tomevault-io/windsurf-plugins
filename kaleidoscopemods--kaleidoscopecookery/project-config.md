---
trigger: always_on
description: - 除非我明确要求英文，否则不要切换英文叙述。
---

# AGENTS.md

## Language policy

- 默认使用简体中文回答。
- 除非我明确要求英文，否则不要切换英文叙述。
- 代码、命令、报错、API 名称保持原文，不要强行翻译。
- 提问澄清时也使用中文。

## Project overview

- Repository: `kaleidoscopecookery`
- Platform: Minecraft Forge 1.20.1 mod
- Java version: 17 (`build.gradle`, `gradle.properties`)
- Gradle wrapper: 8.14.3 (`gradle/wrapper/gradle-wrapper.properties`)
- Mod ID: `kaleidoscope_cookery`
- Main package: `com.github.ysbbbbbb.kaleidoscopecookery`
- CI currently builds with `./gradlew build` in `.github/workflows/gradle-publish-1.20.1.yml`

## Instruction file inventory

- No existing root `AGENTS.md` was present before this file was added.
- No `.cursorrules` file was found.
- No `.cursor/rules/` directory was found.
- No `.github/copilot-instructions.md` file was found.
- No extra agent-specific instruction files were discovered in this repo.
- If new instruction files are added later, treat deeper/local files as more specific than this root guide.

## Environment and tooling

- Always use the Gradle wrapper, not a system Gradle install.
- Windows commands: `./gradlew.bat <task>`
- Unix/macOS commands: `./gradlew <task>`
- The build disables the Gradle daemon in `gradle.properties`; expect one-shot daemon startup messages.
- Source encoding is UTF-8 (`tasks.withType(JavaCompile).configureEach`).
- ForgeGradle, Parchment mappings, and Sponge Mixin are configured in `build.gradle`.
- The project includes generated resources from `src/generated/resources`.
- The default Forge run directories are under `run/`, which is gitignored.

## Repository layout

- `src/main/java/` — main Java sources
- `src/main/resources/` — handwritten resources, mixin config, `mods.toml`, assets, data
- `src/generated/resources/` — generated resources included in the main resource set
- `src/test/` — not present at the time of writing
- `src/gameTest/` — not present at the time of writing
- `.github/workflows/gradle-publish-1.20.1.yml` — CI build workflow
- `build.gradle` — source of truth for runs, dependencies, mappings, mixins, and resource handling

## Verified commands

These commands were checked directly in this repository.

### Core build commands

- List all tasks: `./gradlew.bat tasks --all`
- Build the mod: `./gradlew.bat build`
- Clean outputs: `./gradlew.bat clean`
- Build help: `./gradlew.bat help --task build`

Notes:

- `build` completed successfully in this repo.
- `build` currently runs `test`, but `test` resolves to `NO-SOURCE` because no committed test sources were found.
- `assemble` exists and is narrower than `build` if you only need packaging.

### Forge run tasks

- Client run: `./gradlew.bat runClient`
- Second client profile: `./gradlew.bat runClient2`
- Dedicated server run: `./gradlew.bat runServer`
- Data generation: `./gradlew.bat runData`
- GameTest server task: `./gradlew.bat runGameTestServer`

These task names were verified with `help --task` and `tasks --all`.

### Test commands and single-test guidance

- Gradle exposes a `test` task: `./gradlew.bat test`
- Gradle also exposes single-test filtering syntax: `./gradlew.bat test --tests "com.example.MyTest"`
- However, this repo currently has no committed `src/test/java` tree, no discovered JUnit test classes, and `test` currently reports `NO-SOURCE`.
- Practical meaning: there is no usable single-test workflow today because there are no committed tests to target.
- If a proper JVM test suite is added later, prefer `--tests` for single-class or single-method runs.

### Command selection rules

- For general validation, use `build`.
- For content/datagen changes, run `runData` and review the generated diff.
- For client-only behavior, rendering, tooltips, or visual behavior, run `runClient`.
- For common/server-side logic, registries, recipes, loot, entities, or worldgen, prefer `runServer` or `build`.
- Use `runGameTestServer` only if real GameTests are added; none were found during this analysis.
- If you are unsure what exists, start with `tasks --all`.

## Current validation reality

- No dedicated lint task was found.
- No Checkstyle, Spotless, or EditorConfig file was found.
- No committed unit test source tree was found.
- No committed GameTest source tree was found.
- CI currently verifies `build`; do not claim stronger automated coverage than that.
- The build does emit existing compiler warnings; do not treat those warnings as newly introduced unless your change causes them.

## Code organization patterns

- The mod entrypoint is `KaleidoscopeCookery` and wires registries onto the Forge mod event bus.
- Registry-heavy code lives under `init/` with classes like `ModItems`, `ModBlocks`, `ModEffects`, and `ModRecipes`.
- Event handlers are placed under `event/` and commonly use `@Mod.EventBusSubscriber` plus static `@SubscribeEvent` methods.
- Datagen code lives under `datagen/`, with providers registered from `DataGenerators`.
- Mixin classes live under `mixin/` and must stay synchronized with `src/main/resources/kaleidoscope_cookery.mixins.json`.
- Utilities are collected under `util/` and tend to be small, static helper methods.

## Code style guidelines

### Formatting

- Use 4-space indentation.
- Keep opening braces on the same line.
- Match the spacing style already present in the touched file instead of reformatting unrelated code.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KaleidoscopeMods/KaleidoscopeCookery](https://github.com/KaleidoscopeMods/KaleidoscopeCookery) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
