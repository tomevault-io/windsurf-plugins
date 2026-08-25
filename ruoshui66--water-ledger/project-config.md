---
trigger: always_on
description: This repository is a local-first personal finance ledger. Treat real bills,
---

# Water Ledger Agent Guide

This repository is a local-first personal finance ledger. Treat real bills,
account names, balances, databases, logs, brokerage credentials, and generated
reports as private user data.

## Repository Skills

This repository includes task-specific skills under `skills/water-ledger-*`.
When a user request matches one of those skills, the agent must open and read
the relevant `SKILL.md` before taking task actions, then follow it together
with this guide.

Use the Water Ledger skills for these tasks:

- First-time setup or "what is this/how do I start": `water-ledger-init`.
- Importing or rebuilding bills: `water-ledger-import`.
- Adding accounts, changing account mappings, or setting balances:
  `water-ledger-add-account`.
- Asset snapshots, brokerage configuration, liabilities, or net-worth curves:
  `water-ledger-assets`.
- Classification, transfer, refund, or cashflow rule changes:
  `water-ledger-rules`.
- Any commit, publish, release, or public handoff: `water-ledger-privacy-check`.

If more than one skill applies, use the smallest relevant set in the natural
order of the workflow. For example, adding a wallet balance and rebuilding the
dashboard should use `water-ledger-add-account` before `water-ledger-import`;
publishing afterward should also use `water-ledger-privacy-check`.

If a skill file is missing or unreadable, say so briefly and continue with the
best fallback while still following the privacy rules in this guide.

## Product Stance

When a new user asks how to use this project for their own bookkeeping, do not
make them operate the whole pipeline by hand. Expose only the decisions and
files that genuinely need the user:

- Which accounts they own and what each account should be called.
- Current balances for wallets, cash, liabilities, or other accounts whose
  statements do not provide balances.
- Exported Alipay, WeChat, bank, brokerage, or manual transaction files.
- Optional brokerage API credentials or snapshots.

The agent should do the rest: initialize the workspace, create private
directories, edit local config from the user's answers, place supplied files in
the correct import folders, run imports, start the local dashboard, and explain
only the next human action.

Open-source defaults must stay minimal. A fresh `private/config.yaml` should
start with only:

- 主银行卡
- 微信余额
- 支付宝余额

If the user mentions additional accounts during the conversation, add them to
`private/config.yaml` for the user, then run `python -m water_ledger import` so
the database and dashboard pick them up. Do not add extra personal assumptions
such as brokerage, loans, in-transit funds, or wealth-management accounts to the
public example config.

## Dialog-First User Input

When the agent environment supports a native user-input dialog or modal, use it
before falling back to prose questions. This applies to Codex, Claude Code, and
similar local coding agents.

Use dialogs for user decisions and private values during:

- First-time initialization.
- Adding a new account.
- Recording or refreshing manual balances.
- Choosing which bill exports the user has available.

Do not ask the user to edit YAML when a dialog plus local file edit can handle
the same job. Ask for the smallest useful batch of information, then update
`private/config.yaml` yourself.

For first-time initialization, collect:

1. Whether to keep the default accounts: 主银行卡, 微信余额, 支付宝余额.
2. Current balances for 微信余额 and 支付宝余额, or "skip for now".
3. Whether the user has Alipay, WeChat, bank, brokerage, or manual bills ready.

For a new account, collect:

1. Account display name and account type.
2. Institution, currency, and whether it counts in net worth.
3. Optional current balance and balance time.

If the account type is `brokerage`, also ask whether the user has historical
net-worth data to import. Accept any date range. Prefer configuring and running
`python -m water_ledger brokerage-history --provider <provider> --start <date>
--end <date> --rebuild` so the project batch-fetches history locally. Use CSV
under `private/imports/brokerage/` only as a fallback or as the script output.
Explain that current balance alone cannot reconstruct past daily market moves;
historical snapshots are needed for an accurate historical curve.

When initializing or adding a brokerage account, proactively set up ongoing
daily net-worth snapshots after the account/provider configuration is in place,
unless the user explicitly says not to. Ask for a preferred daily time only if
the user has not already given one; otherwise use 04:01 local time. If the
brokerage credentials or provider setup are not ready, tell the user what is
missing and leave the schedule step pending rather than silently skipping it.

Use the current agent's native recurring-task feature by default when it is
available:

- In Codex app, create or update a Codex automation.
- In Claude Code, create or update a Claude routine/cron task.

The recurring task should run the same project-level snapshot command from this
repository: `.venv/bin/python -m water_ledger brokerage-snapshot --provider
enabled` when a virtualenv exists, otherwise `python -m water_ledger
brokerage-snapshot --provider enabled`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RuoShui66/Water_Ledger](https://github.com/RuoShui66/Water_Ledger) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
