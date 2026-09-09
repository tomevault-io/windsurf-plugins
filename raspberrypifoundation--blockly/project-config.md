---
trigger: always_on
description: Most subdirectories here are self-contained plugins published to npm — fields, themes,
---

# packages/plugins — first-party plugins

Most subdirectories here are self-contained plugins published to npm — fields, themes,
and workspace add-ons that an app developer loads into a Blockly workspace. Most are
TypeScript, and new ones must be; the remaining JavaScript plugins are legacy.

The rest are tooling, test fixtures, and example apps that happen to live alongside them.
Check the table below before assuming a directory is a plugin.

Repo-wide conventions (commits, licence headers, naming) are in the
[root `AGENTS.md`](../../AGENTS.md) and are not repeated here.

## Layout

Every actual plugin follows the same shape:

```
packages/plugins/<name>/
  src/
    index.ts        # public entry point (or index.js for JS plugins)
  test/
    *.mocha.js      # unit tests — the suffix is required, see below
    index.html      # playground page served by `npm start`
    index.ts        # playground setup
  package.json
  tsconfig.json     # present only for TypeScript plugins
  README.md
```

## Directories that are not plugins

Several packages live here for convenience but are not plugins you load into a workspace.
Don't treat them as examples of how a plugin should look:

| Directory                       | Package                   | What it actually is                                                     |
| ------------------------------- | ------------------------- | ----------------------------------------------------------------------- |
| `dev-scripts/`                  | `@blockly/dev-scripts`    | The `blockly-scripts` CLI that every plugin's npm scripts call          |
| `dev-tools/`                    | `@blockly/dev-tools`      | Shared playground and test helpers that plugins import                  |
| `dev-create/`                   | `@blockly/create-package` | The scaffolding generator for new plugins                               |
| `block-test/`                   | `@blockly/block-test`     | Test blocks used by our own test suites — not blocks for app developers |
| `migration/`                    | `@blockly/migrate`        | A CLI that migrates apps to newer versions of Blockly                   |
| `sample-app/`, `sample-app-ts/` | private, unpublished      | Example applications                                                    |

`block-test` deserves particular care: `packages/blockly` takes it as a devDependency and
loads it from the Mocha setup, so editing those blocks can break the **core** test suite,
not just this package's.

## Commands

Run these from inside a plugin's directory:

```bash
npm start        # webpack dev server serving test/index.html, with hot reload.
                 # The port is chosen automatically and printed on startup.
npm test         # bundle test/*.mocha.js with webpack, then run them under Mocha
npm run build    # production webpack build into dist/
npm run clean
npm run lint
```

All of these shell out to `blockly-scripts`, the shared CLI in `dev-scripts`. If a plugin
needs custom Mocha settings, add `test/.mocharc.js` and it will be used instead of the
default.

To run a target from the repo root instead, use Nx. **The Nx project name is the `name`
field in the plugin's `package.json`, which is often not the directory name.** Package
names follow a type-based convention, so a generic plugin in `modal/` publishes as
`@blockly/plugin-modal`:

| Plugin type     | Package name                              |
| --------------- | ----------------------------------------- |
| Field           | `@blockly/field-*`                        |
| Theme           | `@blockly/theme-*`                        |
| Block           | `@blockly/block-*` or `@blockly/blocks-*` |
| Block extension | `@blockly/extension-*`                    |
| Workspace       | `@blockly/workspace-*`                    |
| Anything else   | `@blockly/plugin-*`                       |

Look the name up rather than guessing it from the directory:

```bash
npx nx show projects            # list every project name
npx nx run @blockly/field-slider:test
```

The full convention, including the tags to put in `package.json`, is in
[the plugin naming guide](../docs/docs/guides/contribute/core/plugins/naming.mdx).

## Plugin-specific conventions

- **Test files must be named `*.mocha.js`.** `blockly-scripts test` looks only for that
  pattern; if it finds nothing it prints a warning and exits **0**. A misnamed test file
  therefore fails silently and CI stays green, so double-check the suffix when adding
  tests.
- **Plugin tests are JavaScript.** Core takes TypeScript tests, but the suffix match
  above is literal, so a `.mocha.ts` file here is skipped without an error. Write plugin
  tests as `.mocha.js` until that changes.
- Tests use the `tdd` UI (`suite` / `test`), not BDD.
- **Plugin UI must be accessible.** The same screen reader and keyboard requirements
  apply here as in core, and the utilities to meet them — `Blockly.utils.aria` and the
  keyboard navigation policies — come from core.
  See [DOM and accessibility](../blockly/AGENTS.md#dom-and-accessibility).
- **New plugins are TypeScript.** Some existing ones are JavaScript; those are legacy
  and are not the pattern to copy.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RaspberryPiFoundation/blockly](https://github.com/RaspberryPiFoundation/blockly) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
