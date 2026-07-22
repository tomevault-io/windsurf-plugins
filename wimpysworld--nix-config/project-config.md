---
trigger: always_on
description: This repository is a NixOS, nix-darwin, and Home Manager flake for workstations, servers, VMs, macOS systems, and ISO images.
---

# AGENTS.md

## Scope

This repository is a NixOS, nix-darwin, and Home Manager flake for workstations, servers, VMs, macOS systems, and ISO images.

`AGENTS.md` is the canonical agent instruction file. `CLAUDE.md` only imports it; keep vendor-specific shims thin and do not duplicate rules there.

Preserve runnable commands, Nix module patterns, registry semantics, and tool-specific configuration. Never delete files without explicit confirmation.

## Reference tools

Use the NixOS MCP server as the primary reference for NixOS, Home Manager, nix-darwin options, packages, modules, versions, and flakes. Do not rely on training data when an MCP lookup can answer the question.

| Purpose                | Tools                                                                                                                    |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| NixOS options/packages | `mcp__nixos__nixos_search`, `mcp__nixos__nixos_info`                                                                     |
| Home Manager options   | `mcp__nixos__home_manager_search`, `mcp__nixos__home_manager_options_by_prefix`, `mcp__nixos__home_manager_list_options` |
| nix-darwin options     | `mcp__nixos__darwin_search`, `mcp__nixos__darwin_options_by_prefix`, `mcp__nixos__darwin_list_options`                   |
| Package versions       | `mcp__nixos__nixhub_package_versions`, `mcp__nixos__nixhub_find_version`                                                 |
| Flake search           | `mcp__nixos__nixos_flakes_search`                                                                                        |

## Commands

Run `just --list --unsorted` before inventing commands.

| Task                          | Command                                                                                                                                  |
| ----------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| Format and lint Nix files     | `just format [paths]` or `nix fmt`                                                                                                       |
| Validate registries           | `just lint-registry`                                                                                                                     |
| Evaluate flake and configs    | `just eval`                                                                                                                              |
| Run flake checks              | `just check`                                                                                                                             |
| Build all configs             | `just build`                                                                                                                             |
| Build one host or Home config | `just build-host hostname=<host>`; `just build-home username=<user> hostname=<host>`                                                     |
| Build a package               | `just build-pkg pkg=<pkg> hostname=<host>`                                                                                               |
| Switch or apply configs       | `just switch`; `just apply`; `just apply-home`; `just apply-host`                                                                        |
| Update flake inputs           | `just update`                                                                                                                            |
| ISO and install               | `just iso`; `just inject-tokens remote=<host> user=nixos`; `just install host=<host> remote=<target> keep_disks="false" vm_test="false"` |

Run `just eval` before finishing Nix changes. Run `just lint-registry` after editing registry TOML.

## Nix style

- British English spelling; comments use full sentences with proper punctuation.
- Format with `nixfmt` via `nix fmt` or `just format`.
- Prefer `lib.mkDefault` and `lib.mkForce` over plain values for overridability.
- Use `lib.optional` and `lib.optionals` for conditional lists.
- Use explicit `inherit` statements for clarity.
- Use string interpolation inside strings: `"${variable}"`.
- Use camelCase for Nix attributes and functions; kebab-case for files and directories.
- Hostnames follow themes: Sith Lords for workstations/servers, TIE fighters for VMs.

## Layout

- `common/default.nix` - shared cross-platform config imported by NixOS and nix-darwin.
- `nixos/{hostname}/`, `darwin/{hostname}/` - host-specific configs, disks, kernel modules.
- `_mixins/` directories - self-contained modules with a `default.nix` entry point.
- `nixos/_mixins/` - system-level services, kernel, boot, networking, system packages.
- `home-manager/_mixins/` - user programs, dotfiles, user scripts, Home packages.
- `pkgs/` - custom packages exposed through the local overlay; register new packages in `pkgs/default.nix`.
- `.github/workflows/` - CI behaviour and auto-merge gates.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wimpysworld/nix-config](https://github.com/wimpysworld/nix-config) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
