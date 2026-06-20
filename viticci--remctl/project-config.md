---
trigger: always_on
description: Use when an agent needs to read, create, edit, complete, inspect, or troubleshoot Apple Reminders through the RemCTL CLI on macOS.
---


# RemCTL

RemCTL is a power-user Apple Reminders CLI. It reads the local Reminders CoreData database for fast, detailed output and writes normally through `remctl-bridge` using EventKit. Unsupported private metadata writes are available only when explicitly requested with `--private`; those go through `remctl-private` and Apple's private ReminderKit APIs. It is CLI-only: there is no local API server, token, launch agent, or service command.

## Default Workflow

- Use the installed command for user tasks: `remctl ...`.
- Prefer JSON for automation and verification: `remctl today --json`, `remctl show Work --json`, `remctl info <id> --json`.
- Never write directly to the Reminders SQLite database.
- Do not use `--via-eventkit` by default. It is a limited read-only fallback for `show`, `search`, `today`, and `upcoming` when Full Disk Access blocks a basic read and the task can tolerate missing Reminders metadata.
- For private reminder metadata, use regular `add` or `edit` with `--private`; for private list appearance, Groceries mode, or regular/smart-list pin state, use `list-create --private`, `list-edit --private`, `list-pin --private`, or `list-unpin --private`; for list groups, use `group-create`, `group-edit`, `list-create --private --group`, or `group-delete` with `--private`; for custom smart lists, use `smart-list-create`, `smart-list-edit`, or `smart-list-delete` with `--private`; for Reminders templates, use `template-create`, `template-apply`, or `template-delete` with `--private`. Do not use raw database mutation.

## Agent Routing

Start by deciding the write path. Public EventKit writes are stable and do not need `--private`. Private ReminderKit writes are unsupported, explicit, and required for Reminders-only metadata that EventKit cannot save.

| User intent | Command path | Private? | Verify with |
| --- | --- | --- | --- |
| Read due items, lists, groups, reminders, tags, sections, subtasks | `today`, `upcoming`, `overdue`, `lists`, `groups`, `group-info`, `show`, `search`, `info`, `tags`, `sections`, `subtasks` | No | same command with `--json` |
| Create/edit ordinary reminder fields | `add`, `edit`, `done`, `undone`, `delete` | No | `info <id> --json` or `show <list> --json` |
| Due date, priority, notes, recurrence, EventKit alarm | `add` or `edit` with `-d`, `-p`, `-n`, `--recurrence`, `--alarm` | No | `info <id> --json`; recurrence appears as `recurrence` |
| Move an existing reminder to another list | `edit <id> -l LIST` or `edit <id> --list-id ID` | No | `info <id> --json` or `show <destination> --json` |
| Synced rich URL, real tags, section, shared-list assignment, subtask, image, real flag, urgent, Early Reminder, location alarm | `add --private` or `edit --private` | Yes | `info <id> --json`; UI/device check when sync matters |
| List appearance, Groceries metadata, list or smart-list pin state | `list-create --private`, `list-edit --private`, `list-pin --private`, `list-unpin --private` | Yes | `lists --json` for list color/badge/Groceries/pin state, `smart-lists --json` for smart-list appearance/pin state |
| List group create/rename/membership/order/delete | `group-create`, `group-edit`, `list-create --private --group`, `group-delete` | Yes | `group-info <group> --json`, `groups --json`, `lists --json`, and `show <group-or-child-list> --json` |
| Custom smart list create/edit/delete | `smart-list-create`, `smart-list-edit`, `smart-list-delete` | Yes | `smart-lists --json` |
| Saved Reminders templates | `templates`, `template-info`, `template-create`, `template-apply`, `template-delete` | Reads no; writes yes | `templates --json`, `template-info`, then `show <new list> --json` after apply |

## Limited EventKit Read Fallback

`--via-eventkit` is not normal RemCTL output. Use it only when a supported basic read command is blocked by Full Disk Access and the user request does not need full Reminders fidelity.

Supported commands:

```bash
remctl show Work --via-eventkit --json
remctl search "query" --via-eventkit --json
remctl today --via-eventkit --json
remctl upcoming 7 --via-eventkit --json
```

Fallback JSON is a wrapper object with `source: "eventkit"`, `fidelity: "limited"`, `idWarning`, and `items`. Each item has `eventKitId`, not RemCTL numeric `id`. Never pass `eventKitId` to `info`, `edit`, `done`, `delete`, `link`, `open`, `subtasks`, or any command that expects a numeric ID.

Unavailable in this mode: RemCTL numeric IDs, `--list-id`, table output, sections, synced tags, private rich links, urgent state, template internals, smart-list internals, and exact ID compatibility. If the task needs any of those, fix Full Disk Access for the host process instead of using the fallback.

High-value guardrails:

- Do not use `--private` for normal recurrence or normal `--alarm`; those are EventKit features.
- Do use `--private --early-reminder` for Reminders' Early Reminder menu values; this is separate from EventKit alarms.
- Location alarms use the `edit --private --location-*` guardrail but are saved through the EventKit bridge as structured-location alarms; verify them in `info --json` under `alarms`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [viticci/remctl](https://github.com/viticci/remctl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
