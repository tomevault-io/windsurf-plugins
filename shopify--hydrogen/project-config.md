---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Changeset Rules

Changesets track what packages need new releases and at what version bump level. They can be added via `pnpm changeset add` or by manually creating a markdown file with proper YAML frontmatter (either method is fine).

If changes affect `packages/*/src/**` or `packages/*/package.json`, a changeset is required.

These changesets go into Hydrogen's changelog, and are also used to generate upgrade instructions. **Write the changeset as if the audience is a merchant building with Hydrogen.**

### Rule 1: Skeleton Template Changes

Any change to `templates/skeleton` must include a changeset specifying a **patch** bump for **both** `@shopify/cli-hydrogen` and `@shopify/create-hydrogen` (in addition to `skeleton` itself). See the [Quick Reference](#quick-reference-contributing-to-skeleton-or-cli) below for details.

If forgotten: new scaffolded projects will use a stale template until someone catches the gap.

### Rule 2: hydrogen-react Changes

The entire contents of `hydrogen-react` are re-exported in Hydrogen. Any changeset for `hydrogen-react` must also specify a corresponding bump for `hydrogen`.

If forgotten: Hydrogen consumers will not get the `hydrogen-react` update until a separate Hydrogen release happens to include it.

For the full release process (standard, back-fix, snapshot, failure recovery), see the `hydrogen-release-process` skill. For versioning semantics, see the `hydrogen-versioning` skill.

### CLI Dependency Graph
The Hydrogen CLI system works through a plugin architecture:

```
Developer's Project (e.g., skeleton template)
    ├── package.json
    │   └── devDependencies
    │       └── @shopify/cli
    │
    └── npm scripts
        └── "shopify hydrogen dev" commands
                    ↓
            @shopify/cli (main CLI)
                    ↓
            @shopify/cli-hydrogen (plugin)
                    ↓
            Hydrogen commands available
```

**How it works:**
- **@shopify/cli**: The main Shopify CLI installed in project's devDependencies
- **@shopify/cli-hydrogen**: Plugin package that adds `hydrogen` subcommands
- When `@shopify/cli` detects `@shopify/cli-hydrogen` in dependencies, it loads the plugin
- This enables commands like `shopify hydrogen dev`, `shopify hydrogen build`, etc.
- The plugin uses oclif framework for command structure and hooks

**Example flow:**
1. Developer runs `npm run dev` in their project
2. This executes `shopify hydrogen dev --codegen`
3. `@shopify/cli` receives the command and delegates to `@shopify/cli-hydrogen`
4. The hydrogen plugin executes the dev server with MiniOxygen

## Skeleton Template and Project Scaffolding

### Skeleton Template Location
The skeleton template is the default starter template for new Hydrogen projects:
- Located at `templates/skeleton/` in the Hydrogen repo
- Serves as the foundation for `npm create @shopify/hydrogen@latest`
- Includes both TypeScript configuration (default) and can be transpiled to JavaScript

### How Project Scaffolding Works

When developers run `npm create @shopify/hydrogen@latest`:

1. **Default behavior**: Uses the skeleton template bundled inside `@shopify/create-hydrogen`
   - No network fetch required—the template is pre-bundled at build time
   - This is why `create-hydrogen` must be bumped when skeleton changes

2. **Custom templates** (`--template` flag): Downloads from GitHub
   - Uses GitHub API to fetch the specified template
   - Supports community templates and alternative starters

<details>
<summary>Technical Details: The Bundling Chain</summary>

During a Hydrogen release, templates are bundled through this chain:

```
templates/skeleton/
    ↓ bundled into
@shopify/cli-hydrogen (dist/assets/templates/)
    ↓ bundled into
@shopify/create-hydrogen (bundles cli-hydrogen at build time)
    ↓ published to npm
npm create @shopify/hydrogen@latest
```

The `dist` branch also receives compiled templates for alternative distribution methods.

</details>

### Quick Reference: Contributing to Skeleton or CLI

#### I'm Updating the Skeleton Template

**Required changeset packages:**
- `skeleton` — the source you changed
- `@shopify/cli-hydrogen` — bundles skeleton into its dist
- `@shopify/create-hydrogen` — bundles cli-hydrogen into its dist

⚠️ **Important**: When you run `pnpm run changeset add`, it only shows packages with
actual code changes. You must **manually select** cli-hydrogen and create-hydrogen
even though you didn't change their code. Alternatively, manually add those lines
to the changeset file after creation.

**Example changeset:**
```md
---
"skeleton": patch
"@shopify/cli-hydrogen": patch
"@shopify/create-hydrogen": patch
---

Update skeleton template with [description of your changes]
```

**Canonical example**: See [PR #3232](https://github.com/Shopify/hydrogen/pull/3232) for a complete skeleton update with proper changeset.

<details>
<summary>Why all three packages?</summary>


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Shopify/hydrogen](https://github.com/Shopify/hydrogen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
