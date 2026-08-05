---
trigger: always_on
description: > **This file is the single source of truth for all AI agent instructions in this
---

# AGENTS.md — koni-ea

> **This file is the single source of truth for all AI agent instructions in this
> project.** Cursor, Gemini, Codex CLI, Copilot CLI, and Claude Code all read it.
> [`CLAUDE.md`](CLAUDE.md) is a thin pointer back here plus a Koni-Docs config
> block; on any conflict, AGENTS.md wins.

## What this repo is

`koni-ea` gives partners, customers, and users of **Senti** everything needed to
build a trading bot and deploy it — an agent skill set that encodes the MQL5
standard, and a working template to copy.

Senti is a trading-bot platform. It runs bots on **its own** MT5 terminals — Windows
VPS hosts in a datacenter near the broker, running 24/5 — logged into the broker
account the user linked. A bot reaches it as a compiled `.ex5` plus its `.set` preset,
uploaded to the user's private catalog.

### The framing you must never get wrong

**The user writes MQL5 source. Senti compiles it and runs it.**

They paste `.mq5` into Senti's **Author Studio**, press Compile, press Save as EA, and
deploy. Senti runs a static safety scan, compiles headlessly on its own build host,
and builds the preset from the source's `input` defaults.

Never tell a user they need MetaEditor, an `.ex5`, a `.set` file, or a Windows
machine. **They need a browser.** A local MetaTrader is optional and only useful for
running the Strategy Tester themselves.

Three things follow, and they are worth saying unprompted:

- **The bot goes live when Senti deploys it**, never when it is attached to a local
  chart.
- **Their machine can be off.** Once deployed, nothing on their computer participates.
- **Any local MT5 stays on a demo account.** The same bot on their machine *and* on
  Senti against one broker account doubles every position and leaves each instance
  managing trades the other opened. Silent, and the most expensive mistake available.

**Code that will not pass the safety scan.** Blocked before compiling: any `#import`;
every `WebRequest` (the allowlist ships empty, and a non-literal URL is always
refused); `FileDelete`, `FolderDelete`, `FolderClean`, `FileMove`; `SendFTP`. Do not
write them into a strategy and do not suggest them — design around the restriction.

Full model: [docs/RUNNING-ON-SENTI.md](docs/RUNNING-ON-SENTI.md).

This is a **content/distribution repo**. It has no build step and no app of its
own. The code under `templates/` is a delivered artifact, not something this repo
compiles or runs.

---

## If you were asked to build a bot, start here

This is the most common task in this repo. Follow it in order.

### Step 0 — load the standard

Read [`skills/koni-ea-dev/SKILL.md`](skills/koni-ea-dev/SKILL.md) before writing
any MQL5. It is written to be read by an agent, and it names the non-negotiables.
Load its `references/*.md` on demand as each step needs them — do not read all
seven up front.

### Step 1 — copy the template, do not start from a blank file

```bash
cp -r templates/mql5/STARTER_EA <destination>/MY_STRATEGY
```

[`templates/mql5/STARTER_EA/`](templates/mql5/STARTER_EA/) already implements every
lifecycle, risk and safety mechanic correctly. Starting blank means re-deriving all
of it, and the failure mode is silent: the code runs, the backtest is green, and
the bug only appears on a live account.

### Step 2 — rename to the version layout

`<ALGO>` is `UPPER_SNAKE_CASE`. The folder, all three basenames, and
`#property version` inside the `.mq5` must state the same `X.YY`:

```
MY_STRATEGY/v1/v1.00/
    MY_STRATEGY_v1.00.mq5
    MY_STRATEGY_v1.00.set
    MY_STRATEGY_v1.00.md
    backtest/
```

### Step 3 — replace `Signal()`, and only `Signal()`

Everything else in the template is infrastructure. The signal contract:

- Read **closed** bars only — `[1]` and `[2]`. Never `[0]`; the forming bar
  repaints and no backtest will show you the damage.
- Return `+1` long, `-1` short, `0` stand aside.
- Place no orders and mutate no globals — keep it a pure decision.

Adding an indicator means: create the handle in `OnInit`, check it against
`INVALID_HANDLE`, release it in `OnDeinit`, read it via `BufferValue()`.

### Step 4 — update the artifacts together

The `.set` defaults, the input table in the `.md`, and the `input` declarations in
the `.mq5` are three views of one thing. Change one, change all three.

### Step 5 — verify before claiming done

Your part is done when the source is correct and safety-scan clean. **Compiling is
Senti's job**, so do not claim the code compiles — say it is ready to paste into
Author Studio, and that Compile there is the check.

```bash
./scripts/verify.sh
```

That covers English-only text, link resolution, version identity, handle parity,
closed-bar reads, and the VERSION/CHANGELOG pairing. Then, by hand:

- Walk [`references/mql5-pitfalls.md`](skills/koni-ea-dev/references/mql5-pitfalls.md)
  as a checklist
- Confirm the source contains none of the safety-scan blockers listed above
- Hand off: the user pastes it into Author Studio and presses **Compile**

**Do not claim the code compiles.** You have not compiled it — Senti does that, and
its verdict is the only one that counts.

---

## Rules that are never negotiable

These come from production failures, not taste. Each is a bug that a green
backtest hides. Full reasoning:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Koniverse/Koni-ea](https://github.com/Koniverse/Koni-ea) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-03 -->
