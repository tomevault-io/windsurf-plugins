---
trigger: always_on
description: **Objective:** This document guides AI agents in assisting with questions and tasks related to the Ramble experimentation framework.
---

# AI Agent Context for Ramble

**Objective:** This document guides AI agents in assisting with questions and tasks related to the Ramble experimentation framework.

## What is Ramble?

Ramble is a multi-platform experimentation framework written in Python, designed to increase exploration productivity and improve reproducibility. It helps automate and manage tasks such as:

*   Software installation (often using Spack)
*   Acquiring input files
*   Configuring experiments (e.g., parameter sweeps, scaling studies)
*   Executing experiments
*   Extracting and analyzing results

Ramble works on Linux, macOS, and many supercomputers.

## Ramble Command Line Interface (CLI)

Ramble is primarily controlled through the `ramble` command. Key aspects of the CLI include:

*   You should always examine the command line interface before you execute any ramble commands, as the arguments might change over time.
*   **Getting Help:** Users can get help on any command by using `ramble help` or `ramble help --all` for more detailed information on all commands. Help is also available for subcommands (e.g., `ramble workspace --help`).
*   **Discovering Commands and Depth:** To discover the full command hierarchy and determine its maximum depth, start with `ramble help --all`. For any command that shows `...` in its help text (indicating it has subcommands), run that command with `--help` or `-h` to explore its subcommands (e.g., `ramble workspace --help`). This process can be repeated recursively until no more subcommands are found. The longest chain of commands reveals the maximum depth of the CLI.
*   **Key Commands:**
    *   `ramble workspace create`: To set up a new experiment workspace.
    *   `ramble workspace config`: To manage workspace configurations.
    *   `ramble on`: To execute the experiments defined in the workspace.
    *   `ramble list`: To list available applications, modifiers, etc.
    *   `ramble config`: To manage Ramble's configuration settings.
    *   `ramble repo`: To manage Ramble repositories.
*   **Command Reference:** A full list of commands and their options is available in the [Command Reference](https://ramble.readthedocs.io/en/latest/command_index.html) section of the documentation.

## Ramble Configuration Files

Ramble uses YAML files for configuration, drawing inspiration from Spack's configuration system. Configurations are applied in scopes, with higher precedence scopes overriding lower ones (e.g., user settings override system defaults, workspace settings override user settings).

### Workspace Configuration

*   **Main File:** Each workspace has a primary configuration file located at `$workspace/configs/ramble.yaml`. This file defines the experiments, software, and variables for the workspace.
*   **Structure:** All content within `ramble.yaml` lives under the top-level `ramble:` dictionary.
*   **Detailed Documentation:** [Workspace Configuration File](https://ramble.readthedocs.io/en/latest/workspace_config.html)

### Configuration Sections

Ramble supports various sections within its configuration files. These can exist in the workspace `ramble.yaml`, in separate files within `$workspace/configs/`, or in other configuration scopes (user, site, system). Key sections include:

*   `applications`: Defines the experiments to be generated, including application, workload, and experiment scopes, variables, matrices, etc. See the [Application Section](https://ramble.readthedocs.io/en/latest/configuration_files.html#application-section).
*   `config`: Controls internal Ramble behavior, shell settings, Spack command flags, and upload configurations. See the [Config Section](https://ramble.readthedocs.io/en/latest/configuration_files.html#config-section).
*   `env_vars`: Manages environment variable modifications (set, append, prepend, unset) for experiments. See the [Environment Variables Section](https://ramble.readthedocs.io/en/latest/configuration_files.html#environment-variables-section).
*   `software`: Defines software packages and environments, specifying package manager specs (e.g., Spack specs), compilers, and dependencies. See the [Software Section](https://ramble.readthedocs.io/en/latest/configuration_files.html#software-section).
*   `variables`: Defines key-value pairs used for parameterization and expansion within other configuration sections and templates. See the [Variables Section](https://ramble.readthedocs.io/en/latest/configuration_files.html#variables-section).
*   `variants`: Customizes variants for experiment creation, such as selecting the `package_manager`. See the [Variants Section](https://ramble.readthedocs.io/en/latest/configuration_files.html#variants-section).
*   `modifiers`: Specifies experiment modifiers to be applied to experiments, including mode and target executables. See the [Modifiers Section](https://ramble.readthedocs.io/en/latest/configuration_files.html#modifiers-section).
*   `repos`: Lists paths to repositories containing Application definitions.
*   `modifier_repos`: Lists paths to repositories containing Modifier definitions.
*   **Other Sections:**
    *   `base_application_repos`
    *   `base_class_repos`
    *   `base_modifier_repos`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GoogleCloudPlatform/ramble](https://github.com/GoogleCloudPlatform/ramble) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
