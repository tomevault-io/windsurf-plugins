---
trigger: always_on
description: Use when the user mentions SwitchBot devices, smart-home automation, or asks about controlling lights, locks, curtains, sensors, plugs, or IR appliances (TV/AC/fan). Teaches the agent how to drive the authoritative `switchbot` CLI safely, read user preferences from `policy.yaml`, and respect safety tiers.
---


# SwitchBot skill

You are helping the user control their SwitchBot smart home through the
`switchbot` CLI. This skill tells you **how** to do that safely. It does
not duplicate the CLI's documentation — always query the CLI itself for
ground truth about commands, flags, devices, and capabilities.

---

## Authority chain

The `switchbot` CLI is the single source of truth. When you're uncertain
about anything — a command, a flag, a device state, a device type's
supported actions — run the CLI rather than guessing.

| Question | Authoritative command |
|---|---|
| What can I do (cold start)? | `switchbot agent-bootstrap --compact --json` |
| What commands exist? | `switchbot capabilities --json` |
| What flags does this command take? | `switchbot <cmd> --help --json` |
| What devices does the user have? | `switchbot devices list --json` |
| What's this device doing right now? | `switchbot devices status <id> --json` |
| What can I do with this specific device type? | `switchbot devices describe <id> --json` |
| What scenes are configured? | `switchbot scenes list --json` |
| What's in the user's `policy.yaml`? | `cat ~/.config/openclaw/switchbot/policy.yaml` (or the Windows equivalent) |
| Is my quota OK? | `switchbot --json quota status` |
| Is the setup healthy? | `switchbot doctor --json` |
| What automation rules does the user have? | `switchbot rules list --json` |
| Are the rules valid? | `switchbot rules lint` |
| Is the rules engine running? | `switchbot rules tail --follow` (or `rules list --json` for static state) |
| What past events match a rule? | `switchbot rules replay --since <duration> --dry-run` |
| Where do credentials live? | `switchbot auth keychain describe --json` |
| Move credentials into the OS keychain | `switchbot auth keychain migrate` (the user runs this; you don't) |
| Draft an execution plan from intent | `switchbot plan suggest --intent "..." --device <id> [--device <id>…]` |
| Run a plan with per-step approval | `switchbot plan run <file> --require-approval` |
| Draft an automation rule from intent | `switchbot rules suggest --intent "..." [--trigger mqtt|cron|webhook] [--device <id>…]` |
| Inject a rule into policy.yaml | `switchbot policy add-rule [--dry-run] [--enable]` (reads rule YAML from stdin) |
| Why did a rule fire or get blocked? | `switchbot rules trace-explain --rule <name> --last` (or `--fire-id <id>`) |
| Pre-validate rule effect against history | `switchbot rules simulate <rule.yaml> --since 7d` |

Never invent a deviceId, a command name, or a parameter value. If the
CLI doesn't know about it, refuse and explain — don't paper over it.

---

## Required bootstrap (run this first, every session)

Before you take any action, establish context:

```bash
switchbot agent-bootstrap --compact
```

(The output is always JSON; `--json` is redundant here.)

The response is `{ "schemaVersion": "1.1", "data": { ... } }`, and
`data` carries everything you need to orient yourself without burning
quota:

- `cliVersion` — confirm it matches the skill's `authority.cli` range
- `identity` — product, vendor, API version, documentation URL
- `quickReference` — which commands to reach for in common tasks
- `safetyTiers` — the 5-tier enum (see Safety gates below)
- `nameStrategies` — how to resolve a user's spoken name ("bedroom light")
  to a deviceId (ordered list: `["exact", "prefix", "substring", "fuzzy", "first", "require-unique"]`)
- `profile` — which CLI profile is active
- `quota` — today's usage + remaining budget
- `devices[]` — cached devices with `deviceId`, `type`, `name`, `category`, `roomName`
- `catalog` — summary of device types present in the account, with
  safety tiers and supported commands
- `hints[]` — advisory messages the CLI wants the agent to see (possibly empty array; never null)

If `devices[]` looks stale (e.g. the user says they just added a
device), refresh with `switchbot devices list --json` — that writes
through the local cache.

Then read the user's policy:

```bash
cat ~/.config/openclaw/switchbot/policy.yaml 2>/dev/null || \
cat "$HOME/.config/openclaw/switchbot/policy.yaml" 2>/dev/null || \
cat "$USERPROFILE/.config/openclaw/switchbot/policy.yaml" 2>/dev/null
```

If the file doesn't exist, proceed with defaults from the safety section
below — but tell the user once that they don't have a policy yet and
point them at `switchbot policy new` (requires CLI ≥ 3.7.1).

If the user asks whether their policy file is correct, run:

```bash
switchbot policy validate
```

Exit 0 means the file is valid; any other code means the CLI printed
line-accurate errors — relay those errors to the user rather than
trying to read the YAML yourself.

---

## Resolving a name to a device

When the user says "turn on the bedroom light", resolve the name in this
order (this is what `agent-bootstrap` means by `nameStrategies`):

1. **alias** — if `policy.yaml` maps `"bedroom light"` → `<deviceId>`, use that. **This is the most reliable path.**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chenliuyun/switchbot-skill](https://github.com/chenliuyun/switchbot-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
