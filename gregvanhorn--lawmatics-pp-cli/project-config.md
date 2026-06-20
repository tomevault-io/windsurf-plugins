---
trigger: always_on
description: Every Lawmatics resource the API exposes, plus offline full-text search, custom-field reports, and intake analytics... Trigger phrases: `search lawmatics for ...`, `show me overdue tasks in lawmatics`, `bulk reassign lawmatics tasks`, `lawmatics intake report`, `use lawmatics`, `run lawmatics`.
---


# Lawmatics — Printing Press CLI

## Prerequisites: Install the CLI

This skill drives the `lawmatics-pp-cli` binary. **You must verify the CLI is installed before invoking any command from this skill.** If it is missing, install it first:

1. Install via the Printing Press installer:
   ```bash
   npx -y @mvanhorn/printing-press install lawmatics --cli-only
   ```
2. Verify: `lawmatics-pp-cli --version`
3. Ensure `$GOPATH/bin` (or `$HOME/go/bin`) is on `$PATH`.

If the `npx` install fails before this CLI has a public-library category, install Node or use the category-specific Go fallback after publish.

If `--version` reports "command not found" after install, the install step did not put the binary on `$PATH`. Do not proceed with skill commands until verification succeeds.

Lawmatics-pp-cli mirrors your firm's CRM into a local SQLite database so every contact, matter, note, and interaction is searchable in milliseconds. It adds the bulk operations and analytics the Lawmatics UI doesn't: cross-matter overdue task lists, stage bottleneck reports, referral-source revenue, and ad-hoc custom-field pivots straight to CSV.

## When to Use This CLI

Pick this CLI when you need bulk operations across Lawmatics resources, ad-hoc reports the web UI cannot produce, or fast offline search across contacts and matters. Best for intake teams, firm administrators, and developers building automations on top of Lawmatics.

## Unique Capabilities

These capabilities aren't available in any other tool for this API.

### Local state that compounds
- **`search`** — Sub-100ms offline full-text search across contacts, matters, notes, comments, and interactions including custom fields.

  _Reach for this instead of the API when an agent needs cross-entity search by free text._

  ```bash
  lawmatics-pp-cli search "slip and fall" --json
  ```
- **`bottleneck`** — Median matter dwell-time per intake/pipeline stage, surfacing where deals get stuck.

  _Use to answer 'where are matters dying?' without exporting CSVs._

  ```bash
  lawmatics-pp-cli bottleneck --pipeline intake --json
  ```
- **`overdue`** — Every past-due task across every matter, grouped by assignee and sorted by lateness.

  _Standup-ready workload view the Lawmatics UI cannot produce in one click._

  ```bash
  lawmatics-pp-cli overdue --by assignee --json
  ```
- **`since`** — Everything created or updated in the last N hours/days across all entities.

  _Intake-team standup view: what did we touch yesterday?_

  ```bash
  lawmatics-pp-cli since 24h --json
  ```
- **`cf report`** — Join contacts and matters with arbitrary custom fields and pivot to CSV/JSON.

  _Use when the firm needs ad-hoc analytics the web UI's report builder cannot do._

  ```bash
  lawmatics-pp-cli cf report --fields "Case Value,Referral,Stage" --where stage=signed --csv
  ```
- **`velocity`** — Rolling conversion rate (prospect to signed matter) per referral source with sparklines.

  _Marketing ROI question the API answers only with raw lists._

  ```bash
  lawmatics-pp-cli velocity --by source --window 90d --json
  ```
- **`intake stale`** — Prospects with zero interactions in N days.

  _Surfaces leads that are about to die._

  ```bash
  lawmatics-pp-cli intake stale --days 7 --json
  ```
- **`who-touched`** — Unified chronological feed of every interaction, note, task, and email across all matters for a contact.

  _Use before a re-engagement call or status meeting._

  ```bash
  lawmatics-pp-cli who-touched jane.smith@example.com --json
  ```
- **`revenue`** — Sums invoiced and paid amounts per referral source or practice area.

  _Real marketing-channel ROI in one call._

  ```bash
  lawmatics-pp-cli revenue --by source --window ytd --json
  ```
- **`load`** — Open matters, overdue tasks, and billable hours MTD per attorney.

  _Workload balancing in one command._

  ```bash
  lawmatics-pp-cli load --by attorney --json
  ```
- **`explain`** — Markdown brief summarizing a matter from notes, interactions, and tasks.

  _Use before a status call to brief in 5 seconds._

  ```bash
  lawmatics-pp-cli explain MAT-1234 --json
  ```
- **`pipeline drift`** — Matters that skipped a stage or moved backward through the pipeline.

  _Use to find broken intake processes before clients notice._

  ```bash
  lawmatics-pp-cli pipeline drift --json
  ```

### Bulk operations
- **`bulk reassign`** — Reassign every task matching a filter from one user to another in one command.

  _Use during attorney transitions or PTO coverage instead of clicking through hundreds of tasks._

  ```bash
  lawmatics-pp-cli bulk reassign --from alice@firm.com --to bob@firm.com --filter matter.status=open --dry-run
  ```

### Agent-native plumbing
- **`doctor`** — Health check: orphan tasks, contacts with no email, matters in dead stages, custom fields never used, dormant users.

  _Run quarterly to keep the firm's CRM data clean._

  ```bash
  lawmatics-pp-cli doctor --json
  ```
- **`watch`** — Local daemon that polls and POSTs entity diffs to a URL of your choice.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gregvanhorn/lawmatics-pp-cli](https://github.com/gregvanhorn/lawmatics-pp-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
