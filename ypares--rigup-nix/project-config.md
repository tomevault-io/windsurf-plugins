---
trigger: always_on
description: A **rig** is a project-scoped collection of _riglets_ that provide knowledge and tools for AI agents.
---

# Agent Rig System

## Overview

A **rig** is a project-scoped collection of _riglets_ that provide knowledge and tools for AI agents.

## Core Concepts

### Riglet

A riglet is executable knowledge packaged with its dependencies, as a Nix module:

- **Metadata**: When should this riglet be used, is it production-ready or experimental, etc.
- **Knowledge**: SKILL.md + detailed `references/*.md` files documenting processes and recipes
- **Tools**: Nix packages needed to execute those recipes
- **Configuration**: Settings to adapt tools' behaviour to project context

### Rig

A project-level structure that declares which riglets are active:

- Uses `buildRig` to compose riglet modules
- Builds combined tool environment declaratively
- Exposes riglets' tools and documentation

### rigup

A Nix library and CLI tool: http://github.com/YPares/rigup.nix

#### rigup Nix library

Main functions:

- `buildRig`: evaluates riglet modules and ensures they comply with the riglet schema used by rigup. Returns the rig as an attrset: `{ toolRoot = <derivation>; meta = { <riglet> = {...}; }; docAttrs = { <riglet> = <derivation>; }; docRoot = <derivation>; home = <derivation>; shell = <derivation>; }`
- `resolveProject`: inspects the `riglets/` folder of a project and its `rigup.toml` to find out which riglets and rigs it defines. It calls `buildRig` for each rig in the `rigup.toml`
- `genManifest`: generates a markdown+XML manifest file describing the contents of a rig, primarily for AI agent's consumption
- `mkRiglib`: creates a set of utility functions to be used to define riglet Nix modules

Defined in `{{repoRoot}}/lib/default.nix`.

#### rigup CLI tool

A Rust app. It provides convenient access to rig outputs, via commands like `rigup build` and `rigup shell`. This tool is meant for **the user** primarily. Agents should not have to call it directly.

Defined in `{{repoRoot}}/packages/rigup`

## Riglet Structure

Riglets are Nix modules with access to `riglib` helpers

### Example Riglet

```nix
# First argument: the defining flake's `self`
# Gives access to `self.inputs.*` and `self.riglets.*`
# Use `_:` if you don't need it
self:

# Second argument: module args from evalModules
{ config, pkgs, lib, riglib, ... }: {
  # Riglet-specific options (optional)
  options.myRiglet = {
    myOption = lib.mkOption {
      type = lib.types.str;
      description = "Example option";
    };
  };

  # Riglet definition
  config.riglets.my-riglet = {
    # Dependency relationship/Inheritance mechanism: if B imports A, then whenever B is included in a rig, A will automatically be included too
    imports = [ self.riglets.base-riglet self.inputs.foo.riglets.bar ... ];
  
    # Tools can be:
    # - Nix packages: pkgs.jujutsu, pkgs.git, etc.
    # - Script paths: ./scripts/my-script (auto-wrapped as executables)
    tools = [
      pkgs.tool1
      pkgs.tool2
      ./scripts/helper-script  # Becomes executable "helper-script"
    ];

    # Metadata for discovery and context
    meta = {
      description = "What this riglet provides";
      mainDocFile = "SKILL.md"; # Where to start reading the docs (SKILL.md by default)
      intent = "cookbook"; # What the agent should expect from this riglet
      whenToUse = [
        # When the AI Agent should read/use this riglet's knowledge, recipes and tools
        "Situation 1" # or 
        "Situation 2" # or
        ...
      ];
      keywords = [ "keyword1" "keyword2" ];
      status = "experimental"; # Maturity level
      version = "x.y.z"; # Semantic version of riglet's interface (configuration + provided methods, procedures, docs...)
      disclosure = lib.mkDefault "lazy" # How much to show about riglet in manifest
        # mkDefault makes it possible for end users to override this in their rigup.toml
    };

    # Documentation file(s) (Skills pattern: SKILL.md + references/*.md)
    docs = riglib.writeFileTree {
      "SKILL.md" = ...;  # A main documentation file
      references = {       # Optional. To add deeper knowledge about more specific topics, less common recipes, etc.
                           # SKILL.md MUST mention when each reference becomes relevant
        "advanced.md" = ...;
        "troubleshooting.md" = ...;
      };
    };
    # Files can be defined either as inlined strings or nix file derivations/paths.
    # Folders can be defined either as nested attrsets or nix folder derivations/paths,
    # so if you have a ready to use folder you can do:
    #docs = ./path/to/skill/folder;

    # Configuration files (optional) for tools following the
    # [XDG Base Directory Specification](https://specifications.freedesktop.org/basedir/latest/)
    configFiles = riglib.writeFileTree {
      # Built from a Nix attrset
      myapp."config.toml" = riglib.toTOML {
        setting = "value";
      };
      # Read from existing file
      myapp."stuff.json" = ./path/to/stuff.json;
      # Inlined as plain text
      myapp."script.sh" = ''
        #!/bin/bash
        echo hello
      '';
    };

    # EXPERIMENTAL: Prompt commands (slash commands for harnesses like Claude Code)
    promptCommands.my-cmd = {
      template = "Do something specific with $ARGUMENTS";
      description = "What this command does";

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [YPares/rigup.nix](https://github.com/YPares/rigup.nix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
