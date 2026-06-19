---
trigger: always_on
description: Every MultiMail feature from the terminal, plus inbox health, stale-thread detection, and offline search no other MultiMail tool has. Trigger phrases: `check my multimail inbox`, `send email via multimail`, `multimail health check`, `how many emails pending oversight`, `multimail trust status`, `use multimail`, `run multimail`.
---


# MultiMail — Printing Press CLI

## Prerequisites: Install the CLI

This skill drives the `multimail-pp-cli` binary. **You must verify the CLI is installed before invoking any command from this skill.** If it is missing, install it first:

1. Install via the Printing Press installer:
   ```bash
   npx -y @mvanhorn/printing-press install multimail --cli-only
   ```
2. Verify: `multimail-pp-cli --version`
3. Ensure `$GOPATH/bin` (or `$HOME/go/bin`) is on `$PATH`.

If the `npx` install fails (no Node, offline, etc.), fall back to a direct Go install (requires Go 1.23+):

```bash
go install github.com/mvanhorn/printing-press-library/library/other/multimail/cmd/multimail-pp-cli@latest
```

If `--version` reports "command not found" after install, the install step did not put the binary on `$PATH`. Do not proceed with skill commands until verification succeeds.

A Go CLI that matches all 47 MCP server tools as shell commands, adds a local SQLite cache with FTS5 search, and introduces compound commands that surface compliance insights the API alone cannot answer. Agent-native by default: auto-JSON when piped, --compact for token-efficient output, typed exit codes for scripting.

## When to Use This CLI

Use the MultiMail CLI when you need email operations from a shell environment — CI/CD pipelines, Codex tasks, agentic shells, or automation scripts. The CLI excels at batch operations (sync + search + health check in one pipeline), offline analysis (stale threads, inbox health after a single sync), and compliance monitoring (oversight dashboard, trust ladder status). Prefer the MCP server when operating inside a chat-based agent that supports MCP natively.

## Unique Capabilities

These capabilities aren't available in any other tool for this API.

### Compliance observatory

- **`health`** — Single-number composite score combining unread ratio, response time, bounce rate, and quota headroom — instantly tells you if an inbox needs attention.

  _When managing multiple agent mailboxes, reach for this to triage which inbox needs attention first without checking each one individually._

  ```bash
  multimail-pp-cli health --mailbox primary --json
  ```
- **`stale`** — Find conversation threads that have gone unanswered past a configurable threshold — surface the emails you're dropping the ball on.

  _Before composing new emails, check stale threads first — replying to an existing conversation is almost always higher-value than starting a new one._

  ```bash
  multimail-pp-cli stale --days 3 --json
  ```
- **`oversight summary`** — See pending approval count, average decision time, approval rate, and most-gated senders across all mailboxes — the operator's command center.

  _When the operator hasn't checked in, run this to know if the oversight queue is backing up and which senders are triggering the most gates._

  ```bash
  multimail-pp-cli oversight summary --json
  ```
- **`trust status`** — Current trust ladder position per mailbox, what's needed for the next level, and progression history — the agent's autonomy roadmap.

  _Before requesting an oversight upgrade, check your current position and what the operator needs to see before granting more autonomy._

  ```bash
  multimail-pp-cli trust status --json
  ```

### Local state that compounds

- **`quota forecast`** — Predict when your email quota will be exhausted based on rolling send rate — days remaining with confidence interval.

  _Before scheduling a large email batch, check quota forecast to know if you'll hit limits and need to suggest a plan upgrade._

  ```bash
  multimail-pp-cli quota forecast --json
  ```
- **`stats`** — Send/receive volume, top correspondents, peak hours, and delivery rate over a configurable period — understand email patterns at a glance.

  _When planning outreach campaigns or reviewing agent communication patterns, stats gives you the baseline numbers to work from._

  ```bash
  multimail-pp-cli stats --period 30d --json
  ```

### Agent-native plumbing

- **`search`** — Full-text search across cached emails — works without network after sync, searches subjects, bodies, senders, and recipients.

  _When searching for a specific email or topic, use search instead of paginating through inbox results — especially useful in CI/CD where network calls add latency._

  ```bash
  multimail-pp-cli search 'invoice overdue' --json --compact
  ```
- **`sync`** — Cursor-tracked incremental sync of all entities to local SQLite — enables every compound command and offline access.

  _Run sync before any local query to ensure fresh data. After the first full sync, incrementals are fast and cheap._

  ```bash
  multimail-pp-cli sync --full
  ```

## Command Reference

**account** — Manage account

- `multimail-pp-cli account create` — Create a new account
- `multimail-pp-cli account create-challenge` — Request a verification challenge for account creation
- `multimail-pp-cli account create-resendconfirmation` — Resend the account activation email

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [multimail-dev/multimail-cli](https://github.com/multimail-dev/multimail-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
