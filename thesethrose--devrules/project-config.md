---
trigger: always_on
description: Best practices and guidelines for working with shadcn/ui components across various frameworks.
---


# shadcn/ui Best Practices

## Core Philosophy
- Component Ownership / Open Code: Understand that shadcn/ui components are *copied* into your project (typically `components/ui/`) via the CLI, not installed as an opaque dependency. You own this code and are encouraged to customize it.
- Composition: Components are designed to be composable, often built upon lower-level primitives (like Radix UI).
- Tailwind CSS Based: Styling relies heavily on Tailwind CSS utility classes.
- Accessibility: Built with accessibility primitives.
- CLI Driven: Interaction (initialization, adding components, building registries) is primarily done via the `shadcn-ui` CLI.

## CLI Usage

Use a package runner like `npx`, `pnpm dlx`, `yarn dlx`, or `bunx`.

### `init`

Initializes dependencies and configuration (`components.json`, `cn` util, CSS variables) for a project.

Command:
```bash
# Example using npx
npx shadcn-ui@latest init [components...]
```

Usage: `shadcn init [options] [components...]`
Initialize your project and install dependencies.

Arguments:
- `components`: Optional. The components to add or a URL to the component upon initialization.

Options:
- `-y, --yes`: Skip confirmation prompt. (default: true)
- `-d, --defaults`: Use default configuration. (default: false)
- `-f, --force`: Force overwrite of existing configuration. (default: false)
- `-c, --cwd <cwd>`: The working directory. Defaults to the current directory.
- `-s, --silent`: Mute output. (default: false)
- `--src-dir`: Use the src directory when creating a new project. (default: false)
- `--no-src-dir`: Do not use the src directory when creating a new project.
- `--css-variables`: Use CSS variables for theming. (default: true)
- `--no-css-variables`: Do not use CSS variables for theming.
- `-h, --help`: Display help for command.

### `add`

Adds components and their dependencies to your project.

Command:
```bash
# Example using npx
npx shadcn-ui@latest add <component> [component...]
```

Usage: `shadcn add [options] [components...]`
Add a component to your project.

Arguments:
- `components`: The components to add or a URL to the component.

Options:
- `-y, --yes`: Skip confirmation prompt. (default: false)
- `-o, --overwrite`: Overwrite existing files. (default: false)
- `-c, --cwd <cwd>`: The working directory. Defaults to the current directory.
- `-a, --all`: Add all available components. (default: false)
- `-p, --path <path>`: The path to add the component to.
- `-s, --silent`: Mute output. (default: false)
- `--src-dir`: Use the src directory when creating a new project. (default: false)
- `--no-src-dir`: Do not use the src directory when creating a new project.
- `--css-variables`: Use CSS variables for theming. (default: true)
- `--no-css-variables`: Do not use CSS variables for theming.
- `-h, --help`: Display help for command.

### `build`

Generates registry JSON files based on a `registry.json` definition (primarily for building custom component registries).

Command:
```bash
# Example using npx
npx shadcn-ui@latest build [registry]
```

Usage: `shadcn build [options] [registry]`
Build components for a shadcn registry.

Arguments:
- `registry`: Path to `registry.json` file (default: `./registry.json`).

Options:
- `-o, --output <path>`: Destination directory for JSON files (default: `./public/r`).
- `-c, --cwd <cwd>`: The working directory. Defaults to the current directory.
- `-h, --help`: Display help for command.

## Setup & Configuration (`components.json`)
- Initialization: Use the `init` command (see CLI Usage section above) to initialize the project and create the `components.json` configuration file.
- `components.json` is Key: This file dictates CLI behavior, including paths, styling choices, and framework specifics. Ensure it's configured correctly for your project structure *after* initial framework setup (like Tailwind, path aliases).
- Path Aliases: Define and consistently use the correct path aliases specified in `components.json` for importing components (`components`, `ui`), utilities (`utils`, `lib`), and potentially hooks (`hooks`). Ensure these aliases are correctly resolved in your framework's build configuration (`tsconfig.json`, `vite.config.ts`, etc.). Example: `import { Button } from "@/components/ui/button";`
- Styling Configuration (`tailwind` object in `components.json`):
    - Configure the desired `style` (e.g., `default`, `new-york`).
    - Configure the `baseColor`.
    - Set `cssVariables` to `true` (recommended) to enable theming via CSS variables.
    - `css`: Ensure this path points to your actual global CSS file where Tailwind directives (`@tailwind base`, etc.) are defined.
    - `config`: Ensure this path points to your `tailwind.config.js` or `tailwind.config.ts` file.
        - Tailwind v4 Specific: If using Tailwind v4, leave the `tailwind.config` path *empty* (`"config": ""`).
- Framework Integration (`components.json`):
    - `tsx`: Set based on whether your project uses TypeScript.
    - `rsc`: Set based on whether your project uses React Server Components.

## Adding & Using Components
- Use the CLI: Add components using the `add` command (see CLI Usage section above).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TheSethRose) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
