---
trigger: always_on
description: The simvyn skill helps a coding agent operate iOS Simulators, Android Emulators, and supported physical devices through the existing CLI. It covers device selection, app operations, screenshots, device settings, sandbox inspection, logs, and saved Collections. No application SDK integration is needed for these commands; individual operations still depend on the device and build type.
---

# Using simvyn with AI coding agents

The simvyn skill helps a coding agent operate iOS Simulators, Android Emulators, and supported physical devices through the existing CLI. It covers device selection, app operations, screenshots, device settings, sandbox inspection, logs, and saved Collections. No application SDK integration is needed for these commands; individual operations still depend on the device and build type.

The portable skill is an instruction package with a CLI launcher and detailed references. The agent needs a terminal tool and access to your local developer tools. Installing the skill does not add an MCP server, register new tool APIs, or grant additional permissions. The Pi package adds a native extension on top of the same skill.

## Install for Pi

With Pi 0.74 or later installed:

```bash
pi install npm:simvyn
pi list
```

The npm package bundles the CLI and gives Pi three kinds of resources:

| Resource         | Provides                                                                                            |
| ---------------- | --------------------------------------------------------------------------------------------------- |
| Extension        | `simvyn`, `simvyn_screenshot`, `simvyn_logs`, and `simvyn_record` tools, plus the `/simvyn` command |
| Skill            | `/skill:simvyn` and the references linked from it                                                   |
| Prompt templates | `/simvyn-repro`, `/simvyn-screen`, and `/simvyn-logs`                                               |

The extension finds the CLI inside that installation, so a separate global `simvyn` executable is unnecessary:

- `simvyn` runs a CLI command from an argument array with a timeout and truncated output. It rejects `start`, `logs`, `record`, `upgrade`, and argument lists without a subcommand, which would start the dashboard.
- `simvyn_screenshot` captures a booted simulator or Android device and returns the image to the model.
- `simvyn_logs` streams logs for a fixed number of seconds, stops the stream, and saves the full JSON Lines capture to a file.
- `simvyn_record` records the screen for a fixed number of seconds and returns the MP4 path.
- `/simvyn` starts the dashboard in the background and opens it in your browser; `/simvyn stop` and `/simvyn status` manage it. The dashboard keeps running across `/new`, `/resume`, `/fork`, and `/reload`, and stops when Pi exits.

Start a new Pi session and describe the device work, or use a template or the skill:

```text
/simvyn-screen check the login form layout
/skill:simvyn List the available devices and explain which can capture screenshots. Do not change device state.
```

Use `pi config` to turn off the extension, skill, or individual prompt templates. Use `pi install npm:simvyn@<version>` to pin a release, replacing `<version>` with its published version, and `-l` for a project-scoped installation. Pi versions before 0.74 load the skill and prompt templates but not the extension. See the official [Pi package documentation](https://pi.dev/docs/latest/packages) for scope, updates, and removal, and [Pi skills documentation](https://pi.dev/docs/latest/skills) for loading skills.

The npm command installs the published package, not uncommitted changes in this repository. If the current release does not include these resources yet, use the checkout instructions below.

## Install the portable skill for another agent

Install the CLI, then the skill for the agent you use:

```bash
npm install -g simvyn
npx skills add pranshuchittora/simvyn --skill simvyn --agent codex --yes
```

Replace `codex` with a supported agent identifier, such as `claude-code`. The skills CLI installs to the project by default; add `--global` for user-wide scope. Its [official documentation](https://github.com/vercel-labs/skills) lists supported agents and installation options. This command requires the skill to have reached the repository's default branch.

The portable skill installation includes instructions and helper files, not the simvyn application. Keep the entire `skills/simvyn/` directory together if your agent uses a manual skill installation process. Follow that agent's loading instructions and make an installed `simvyn` CLI available on its PATH. Check the CLI version and command help before following instructions from a newer skill.

## Test an unreleased checkout

Use the development Node.js version in the [contributing guide](https://github.com/pranshuchittora/simvyn/blob/main/CONTRIBUTING.md); checkout build requirements can be newer than the installed CLI's runtime minimum. From the repository root, install its locked dependencies and build the release layout:

```bash
npm ci
npm run build:release
```

For a single Pi session, load the checkout's extension, skill, and prompt templates directly, or only its skill with `--skill`:

```bash
pi -e /absolute/path/to/simvyn
pi --skill /absolute/path/to/simvyn/skills/simvyn/SKILL.md
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pranshuchittora/simvyn](https://github.com/pranshuchittora/simvyn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
