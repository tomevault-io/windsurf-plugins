---
trigger: always_on
description: Guidance for coding agents operating in `magi-uI-synthwave`.
---

# AGENTS.md

Guidance for coding agents operating in `magi-uI-synthwave`.

## 1) Repository Scope

- This is a **multi-platform theme workspace**.
- Currently implemented targets: `platforms/jetbrains`, `platforms/zed`, `platforms/opencode`, `platforms/ghostty`, `platforms/vscode`, `platforms/slack`, and `platforms/pi`.
- JetBrains target is an IntelliJ Platform theme plugin (resources-first project).
- Zed target is a file-based theme JSON package (no build toolchain required).
- OpenCode target is a file-based theme JSON package (no build toolchain required).
- Ghostty target is a file-based theme config package (no build toolchain required).
- VS Code target is a manifest + theme JSON package (no build toolchain required).
- Slack target is a plain-text import-string package (no build toolchain required).
- Pi target is a file-based TUI theme JSON package (no build toolchain required).
- There is currently no application runtime code (no Kotlin/Java source trees).

## 2) Rule Sources Discovered

I checked for external agent-rule files and found none in this repo:

- `.cursor/rules/` → not present
- `.cursorrules` → not present
- `.github/copilot-instructions.md` → not present

If any of these files are added later, treat them as higher-priority local guidance.

## 3) Important Paths

- Root workspace README: `README.md`
- Changelog: `CHANGELOG.md`
- JetBrains platform root: `platforms/jetbrains/`
- JetBrains Gradle build file: `platforms/jetbrains/build.gradle.kts`
- JetBrains Gradle properties: `platforms/jetbrains/gradle.properties`
- JetBrains plugin descriptor: `platforms/jetbrains/resources/META-INF/plugin.xml`
- JetBrains theme JSON: `platforms/jetbrains/resources/theme/magi-ui-synthwave.theme.json`
- JetBrains editor scheme XML: `platforms/jetbrains/resources/magi-ui-synthwave.xml`
- Zed platform root: `platforms/zed/`
- Zed platform README: `platforms/zed/README.md`
- Zed theme JSON: `platforms/zed/themes/magi-ui-synthwave-zed.json`
- OpenCode platform root: `platforms/opencode/`
- OpenCode platform README: `platforms/opencode/README.md`
- OpenCode theme JSON: `platforms/opencode/themes/magi-ui-synthwave-opencode.json`
- Ghostty platform root: `platforms/ghostty/`
- Ghostty platform README: `platforms/ghostty/README.md`
- Ghostty theme file: `platforms/ghostty/themes/magi-ui-synthwave-ghostty`
- VS Code platform root: `platforms/vscode/`
- VS Code platform README: `platforms/vscode/README.md`
- VS Code extension manifest: `platforms/vscode/package.json`
- VS Code theme JSON: `platforms/vscode/themes/magi-ui-synthwave-vscode-color-theme.json`
- Slack platform root: `platforms/slack/`
- Slack platform README: `platforms/slack/README.md`
- Slack theme import strings: `platforms/slack/themes/magi-ui-synthwave-slack.txt`
- Pi platform root: `platforms/pi/`
- Pi platform README: `platforms/pi/README.md`
- Pi theme JSON: `platforms/pi/themes/magi-ui-synthwave.json`

## 4) Build / Lint / Test Commands

Run all commands from repo root unless noted.

### JetBrains: core Gradle commands

```bash
./platforms/jetbrains/gradlew -p platforms/jetbrains help
./platforms/jetbrains/gradlew -p platforms/jetbrains clean
./platforms/jetbrains/gradlew -p platforms/jetbrains buildPlugin
./platforms/jetbrains/gradlew -p platforms/jetbrains verifyPlugin
./platforms/jetbrains/gradlew -p platforms/jetbrains check
```

Notes:

- `buildPlugin` is the key packaging command.
- Build artifact is expected under: `platforms/jetbrains/build/distributions/`.
- `verifyPlugin` is configured in `build.gradle.kts` with recommended IDEs.

### Running tests

There are currently no committed test files in this repo.

When tests are added, use:

```bash
./platforms/jetbrains/gradlew -p platforms/jetbrains test
```

Single test class:

```bash
./platforms/jetbrains/gradlew -p platforms/jetbrains test --tests "com.example.MyThemeTest"
```

Single test method:

```bash
./platforms/jetbrains/gradlew -p platforms/jetbrains test --tests "com.example.MyThemeTest.rendersExpectedPalette"
```

### Lint / validation status

- No dedicated linter task (e.g., ktlint/detekt/eslint) is configured right now.
- Treat `check` plus static file validation as minimum quality gate.

### Zed: validation commands (file-based)

No build/test task is configured for Zed themes. Use JSON parse validation:

```bash
python3 -c "import json; json.load(open('platforms/zed/themes/magi-ui-synthwave-zed.json')); print('Zed JSON validation passed')"
```

### OpenCode: validation commands (file-based)

No build/test task is configured for OpenCode themes. Use JSON parse validation:

```bash
python3 -c "import json; json.load(open('platforms/opencode/themes/magi-ui-synthwave-opencode.json')); print('OpenCode JSON validation passed')"
```

### Ghostty: validation commands (file-based)

No build/test task is configured for Ghostty themes. Use key-presence validation:

```bash
python3 -c "from pathlib import Path; p=Path('platforms/ghostty/themes/magi-ui-synthwave-ghostty'); t=p.read_text(); required=['background =','foreground =','cursor-color =','selection-background =','palette = 0=','palette = 15=']; missing=[k for k in required if k not in t]; print('Ghostty theme validation passed' if not missing else f'Missing keys: {missing}')"
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [magimetal/synthwave-themes](https://github.com/magimetal/synthwave-themes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
