---
trigger: always_on
description: Agentic HIL is the hardware gate. It discovers the project's authoritative configuration outside the repository; `AGENTIC_HIL_CONFIG` may override that location with an absolute path. Mandatory `workspace_root` binds either file to the current project, and mandatory `state_root` pins trusted report and hardware-lease state outside the workspace.
---

# Agentic HIL Agent Instructions

Agentic HIL is the hardware gate. It discovers the project's authoritative configuration outside the repository; `AGENTIC_HIL_CONFIG` may override that location with an absolute path. Mandatory `workspace_root` binds either file to the current project, and mandatory `state_root` pins trusted report and hardware-lease state outside the workspace.

For installation and first-time setup, follow [AI_AGENT_QUICKSTART.md](AI_AGENT_QUICKSTART.md): everything installs user-local without admin rights.

Setup has two scopes. User scope is per user, per machine: all of it under the invoking user's home, invisible to other OS users on the host.

| Command | Scope | Does |
|---|---|---|
| `agentic-hil agent-install --agent <agent>` | user, once per user and agent | agent skill + user-level MCP registration; needs no workspace or config, and finishes even where the config location is refused |
| `agentic-hil init [--agent <agent>]` | project, once per project | writes the workspace's authoritative config, then `doctor` |
| `agentic-hil setup --agent <agent>` | both, in order | first run in one command; later projects for this user need `init` alone |

Neither half rolls back the other: `setup` with `ok: false` but `scopes.user.ok` true left a working agent installed; read that field before reporting nothing was installed.

A project with no configuration answers every tool with `config_file_not_found`. `project_config_create` is the way out: it takes no arguments, generates the configuration for the workspace this server is bound to out of attached hardware, and writes every permission `true` (flashing, reset, resuming a halted target in a debug session, COM and CAN writes, and all three `permissions.allow_config_*` grants) except `allow_raw_debugger_commands` and `allow_mass_erase`, which it writes `false`. Those two are not capabilities you are missing: there is no tool here for either, and while either is true `flash_firmware` on that probe is refused, so a file that granted them was a bench that could not flash. The bench is workable from that file as it stands, flashing included, without anybody editing YAML. Report where it is and what it granted (the result's `permissions` is the list, and `narrowed_permissions` names anything narrowed *beyond* those defaults); say `allow_mass_erase` in particular wherever it is true because a mass erase cannot be taken back, and ask the operator which permissions this bench should not have. Never ask for those two to be turned on: that is the one change that stops flashing. Regenerating it later carries over the permissions of the configuration that server loaded at startup for the entries already in it, and anything the regeneration discovers for the first time arrives at those same defaults. A server does not reload, so a permission you narrowed with `project_config_set` in this session is not in what it loaded and a regeneration now writes that permission back open: one more reason regeneration is not yours to reach for. Never delete or move a configuration to get a different one: regenerating is the operator's call, not yours.

Changing an existing configuration goes the same way: over MCP, never with your own file tools, which `setup` denies on that path for exactly this reason. Two permissions gate it, and the split matters: `permissions.allow_config_description_write` opens what the bench **is** (`target.*`, `debuggers.<n>.probe_id` / `executable` / `interface_cfg` / `target_cfg`, `com_ports.<n>.device` / `baudrate` / `serial_number`, every `can_buses.<n>` field but its permissions), and `permissions.allow_config_permissions_write` opens every permission key (each `permissions:` block, plus the two grants that sit directly on a section, `artifacts.allow_upload` and `debug.allow_all_symbols`). The first can be left open without handing over the second.

**Permissions move one way through `project_config_set`.** You may write `false` into a permission. You may never send any other value for one: not `true` for a permission you never touched, and not `true` for one you set to `false` yourself a moment earlier, which would change nothing and is refused all the same. Such a call is refused as `permission_widening_denied`, whatever key it named: the value is checked, and the permissions present in the document before and after the write are compared as well. Only a person reopens a configuration, and there is no tool that does it; report that and stop.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agentic-hil/agentic-hil](https://github.com/agentic-hil/agentic-hil) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
