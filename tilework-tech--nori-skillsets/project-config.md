---
trigger: always_on
description: All changes must be compatible on MacOS and Linux. Be careful of using bash
---

# Plugin package changes.

All changes must be compatible on MacOS and Linux. Be careful of using bash
tools that may behave differently on different systems.

## Claude Code configuration vs. Nori Plugin package

**IMPORTANT:** When discussing "Claude" or "Claude Code configuration":

- **Almost always** means modifying files in `src/cli/features/claude-code/` (the Nori Plugin package)
- **Rarely** means modifying `~/.claude/` (the installed user configuration)

**Default assumption:** Unless explicitly stated otherwise, "modify Claude configuration", "update skills", "change hooks", etc. refers to modifying the Plugin package source files at `src/cli/features/claude-code/`, NOT the installed configuration at `~/.claude/`.

**Examples:**

- "Change the status line" → Modify `src/cli/features/claude-code/statusline/`

**Note:** No built-in profiles are shipped with the package. Profiles (skillsets) are stored at `~/.nori/profiles/` and are obtained from the registry or created by users.

**Only modify `~/.claude/` when:**

- User explicitly says "my installed configuration" or "~/.claude/"
- Testing changes locally before committing to the package

# Skills documentation.

Skills in `~/.nori/profiles/<skillset>/skills/` are self-explanatory. Do not create docs.md files in skill directories unless the skill is particularly complex and requires additional context files.

# Style guide.

## Functions and named parameters.

All functions except class functions should be arrow functions. All of them
should use named args, even if there is a single parameter. Follow this
pattern throughout:

```ts
const foo = (args: { bar: string; baz: number }) => {
  const { bar, baz } = args;
};
```

To set defaults, use the 'withDefaults' helper found in server/src/utils/defaults.ts or
ui/src/utils/defaults.ts:

```ts
import { withDefaults } from '@/utils/defaults';

const foo = (args: { bar?: string; baz?: number }) => {
  const { bar, baz } = withDefaults(args, { bar: 'hello', baz: 10 });
};
```

## @ Imports.

Always import files from the package root. For example, instead of `../foo/bar`
import using `@/foo/bar`.

## Singleton classes.

Private constructors are fine, but do not have empty private constructors.
Empty constructors should just be deleted.

## Null vs optional types and parameters.

All optional types should also be able to be set to null. Never do:

```
const foo = (args: { bar?: string }) => {}
```

Always do:

```
const foo = (args: { bar?: string | null }) => {}
```

## Null checks.

Always use `if (foo == null)` instead of `if (foo === undefined)` or `if (!foo)`.

## Vue component structure.

All vue components should use single file component syntax, in the following
order:

- script
- template
- style

The script section should further be separated as follows:

- all state and variable declarations are at the top, including any computeds
- all functions and business logic follows
- all watches, mount hooks, etc. are all at the end

## try catch error handling.

Only use try/catch on system boundaries. Allow failures to bubble up in
intermediate calls. Do not have try catch handlers for every function, as this
masks the stack trace where the try catch originates.

## Run `npm run format` and `npm run lint`

After ANY changes to the codebase, ALWAYS run `npm run format` and `npm run lint` to catch and fix lint errors.

<system-reminder>ALWAYS fix failing tests, even if they aren't caused by you</system-reminder>

## Do not re-export imports from other files.

<bad-example>
foo.ts
```
import { Foo } from './foo.ts'
export { Foo }
```
</bad-example>

Whenever functions or variables change locations in a refactor, adjust all callers to point to the new home location instead of leaving behind import-export patterns.

## Critical: How to Close the Loop

After implementation is complete and all tests, linting, and formatting pass,
you MUST verify your changes work end-to-end using one or more of the options
below. Choose the option that matches the area of the codebase you changed.

### Option 1: Non-Interactive CLI Commands

**When to use:** Changes to non-interactive subcommands (`list`, `current`, `dir`, `install-location`, `completion`, `clear`, `logout`) or core CLI infrastructure (argument parsing, command registration, output formatting).

**Steps:**

1. Build the project: `npm run build`
2. Set up the isolated environment:
   ```bash
   source .claude/skills/e2e-testing-skillsets/e2e-setup
   ```
3. Seed minimal test data:
   ```bash
   mkdir -p "$E2E_DIR/.nori/profiles/test-skillset"
   echo '{"name":"test-skillset","version":"1.0.0","type":"skillset"}' > "$E2E_DIR/.nori/profiles/test-skillset/nori.json"
   echo '{"activeSkillset":"test-skillset","version":"1.0.0"}' > "$E2E_DIR/.nori-config.json"
   ```
4. Run non-interactive commands against the isolated environment:
   ```bash
   env NORI_GLOBAL_CONFIG=$E2E_DIR node $SKS list
   env NORI_GLOBAL_CONFIG=$E2E_DIR node $SKS current
   env NORI_GLOBAL_CONFIG=$E2E_DIR node $SKS install-location
   ```
5. Run the specific command you changed and verify its output is correct
6. Tear down: `.claude/skills/e2e-testing-skillsets/e2e-teardown`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tilework-tech/nori-skillsets](https://github.com/tilework-tech/nori-skillsets) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
