---
trigger: always_on
description: **Never create commits, pull requests, or push to any remote.** Do not run `git commit`, `git push`, `git pr`, or any equivalent. Leave all git operations to the user.
---

# AGENTS instructions

## Git Policy

**Never create commits, pull requests, or push to any remote.** Do not run `git commit`, `git push`, `git pr`, or any equivalent. Leave all git operations to the user.

## Validation After Changes

After a change to code under `src/`, always follow these rules to ensure quality while being efficient. For documentation, workflow, metadata, translation, or other non-`src/` changes, do not run `just validate` or `just shexli` unless the task specifically requires that validation.

1.  **Run `just validate`** — type-checks the source, lints, and checks formatting. Fix any reported errors.
2.  **Run `just shexli`** — packages the extension and runs the extensions.gnome.org static analyzer on the generated ZIP. Review every finding. Some `warning` or `manual_review` findings can be false positives or accepted GNOME-review tradeoffs, but they must be called out explicitly; fix any real regression before finishing.
3.  **Run targeted integration tests:**
    - If you modified only **one module**, run only its Shell test (e.g., `just test shell tests/shell/desktop/trayIcons`).
    - If you made **formatting-only changes** (Prettier) and have already passed the tests in a previous turn, you only need to run `just validate` and `just shexli`.
    - If you made **architectural or cross-cutting changes**, run `just toolbox test`.

**IMPORTANT:** Never execute `just test shell` or `just toolbox test` chained with another command using `&&`. Always run tests as a separate standalone turn.

To read only the relevant output from a full test run (pass/fail summary):

```sh
just toolbox test 2>&1 | grep -E "PASS:|FAIL:|Results:"
```

Do not leave a task incomplete if either command reports errors or failures.

## Commands

- **Install deps:** `just deps` — runs `yarn install --immutable`; use once or after changing branches
- **Build:** `just build` — compiles TypeScript and SCSS, copies metadata/schemas, and compiles `.mo` files
- **Package:** `just package production` — packs the production extension as a `.zip` in `dist/target/`
- **Development package:** `just package development` — packs the separate DevTool-enabled development ZIP
- **Inspect packages:** `just package check` — builds both ZIPs and verifies their contents and generated line lengths
- **Install:** `just install` — packages and installs the DevTool-enabled development ZIP
- **Uninstall:** `just uninstall` — disables and removes the extension
- **Run (host):** `just run` — installs development, then launches a DevTool-enabled devkit session
- **Run (toolbox):** `just toolbox run` — packages/installs on the host, then runs GNOME Shell inside the Fedora toolbox
- **Create toolbox:** `just toolbox create` — creates `aurora-shell-devel` from the same Fedora/GNOME image used by CI
- **Remove toolbox:** `just toolbox remove` — delete the toolbox
- **Validate:** `just validate` — runs tsc, ESLint, Prettier check, and Stylelint
- **Shexli:** `just shexli` — packages the extension and runs the extensions.gnome.org static analyzer on the generated ZIP
- **Lint:** `just lint` — runs ESLint only
- **Watch SCSS:** `just watch` — watches `src/styles/` and recompiles on change
- **View logs:** `just logs` — shows recent `aurora` entries from the current boot journal
- **Clean:** `just clean` — removes `dist/`
- **Deep clean:** `just clean all` — removes `dist/` and `node_modules/`
- **Unit tests:** `just test unit` — runs unit tests with Node's test runner
- **Coverage:** `just test coverage` — runs unit tests with coverage report
- **Shell tests:** `just test shell [file-or-directory]` — packages and runs all Shell tests, or a recursive target, on the host
- **DevTool Shell test:** `just test dev` — tests `tests/shell/dev/devTool.test.js` against the development ZIP
- **Shell tests (toolbox):** `just toolbox test [file-or-directory]` — preferred full or targeted execution inside the Fedora toolbox
- **DevTool Shell test (toolbox):** `just toolbox test-dev` — runs the development test inside the toolbox
- **Vagrant VM:** `just vagrant create|run|ssh|remove` — full Arch VM kept for manual GNOME environment testing

### i18n commands

- **Regenerate POT template:** `just i18n pot` — builds, then scans compiled JS (`dist/`) and writes the `.pot` into `dist/` (a build artifact, **not** committed — avoids `POT-Creation-Date` churn). Run this whenever translatable strings are added or removed.
- **Merge new strings into .po files:** `just i18n update` — depends on `i18n pot`; regenerates the template into `dist/` then runs `msgmerge` on every `data/po/*.po` against it. The hand-translated `data/po/*.po` files are the committed source of truth.
- **Compile .mo binaries:** `just i18n compile` — compiles each `po/*.po` into `dist/locale/<lang>/LC_MESSAGES/*.mo`. Called automatically by `just build`.

## Repository Structure

- `src/` — TypeScript source root
  - `extension.ts` — minimal production entry point
  - `extension.dev.ts` — development entry point used only by the development ZIP
  - `core/extensionBase.ts` — shared extension lifecycle; creates the context and delegates to `ModuleManager`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [luminusOS/aurora-shell](https://github.com/luminusOS/aurora-shell) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
