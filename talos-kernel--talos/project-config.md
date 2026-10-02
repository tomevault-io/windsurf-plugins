---
trigger: always_on
description: Guidance for Claude Code (and any other coding agent) working in this repository.
---

# CLAUDE.md — Talos

Guidance for Claude Code (and any other coding agent) working in this repository.

## What this is

An autonomous agent that takes instructions over a chat channel, reasons with a language
model, and executes tools — but only after a deterministic security kernel has ruled on
the action. Full architecture in `README.md`. The agent's name and character live in
`SOUL.md`; durable operating discipline lives in `AGENTS.md`, and stable operator
preferences live in `USER.md`. All three are operator-owned prompt state and reload live.

| | |
|---|---|
| Gate path | `policy.py`, **985 lines** — has to stay readable in one sitting |
| Tools | **33**, every one gated |
| Suites | **2933** collected tests · **263** adversarial · 44 end-to-end |
| Home | <https://talos-agent.ch> · docs at `/docs/` |
| Repository | `talos-kernel/talos` is the public source tree |

⚠️ **`public` is blocked for pushing.** Its push url is a deliberate dead end — the
published state is only ever produced by `scripts/sync-public.sh` into a separate clone.
A direct push from here would carry 108 commits and a real author address across.

## The rule everything rests on

**The model proposes, it does not decide.** The reasoner emits `TOOL_CALL` text, the kernel
judges, the executor performs. Weakening that order destroys the product, no matter how
green the tests are.

In practice:

- **Never put security logic in `tools.py`.** The runners are deliberately dumb. Gate
  (policy) and execution (runner) stay separate.
- **Never trust a field the model could omit.** Targets are derived from the real arguments
  via `TARGET_EXTRACTORS`; a tool without an extractor is `DENY` by construction.
- **Never give the reasoner its own tools.** `DISALLOWED_TOOLS_ARGV` and
  `CLAUDE_ISOLATION_ARGV` in `reasoner.py` are security boundaries, not tuning.
- **Never let the model write the approval text.** It comes from the kernel, so the human
  sees the facts rather than the model's description of itself.
- **Never let a plan carry permission.** An announced sequence (`plan.py`) may set order,
  an abort condition and a per-step check — nothing else. Every step still passes
  `decide()` on its own, and there is deliberately no "approve the plan" path: that would
  be consent to actions nobody had seen yet. A plan may only make a run end *earlier* or
  withhold a confirmation; it may never grant one.
- **Never let a plan check touch the world.** `check_met` reads the receipt of the step
  that just ran and nothing else. A condition that could open files would be a read oracle
  around the kernel. Unknown check vocabulary is dropped, never treated as met — the other
  direction would make invented words a way to fake an acceptance.
- **Never let a delegated run write.** `subagent.ReadOnlyCeiling` turns everything except
  reading into `DENY`, including anything that would need approval — a question reaching
  the operator out of the context it came from is how reflexive clicking starts. A
  subagent is born from model text; it must be able to do *less* than its caller.
- **Never let the browser operate a page.** `browse` renders and reads. Clicking, typing
  and form submission have no derivable target, so they cannot be gated — and a tool
  without a target is `DENY` by construction. Rendering also stays inside the resolver
  cage (`browser.resolver_rules`), so a redirect cannot leave the host `guard_url` checked.
- **Never add a second source of permission.** The autonomy dial and the channel ceiling
  can only tighten. Anything that grants rights next to the kernel reintroduces the exact
  problem capability tokens were built to remove.
- **Never let a channel receive.** Every way in fetches: Telegram long-polls, `mail.py`
  pulls over IMAP. A webhook needs a port the world can reach, which turns an
  outbound-only process into a reachable one — that is why inbound WhatsApp does not
  exist and `WhatsAppChannel.poll()` returns `[]` on purpose rather than "not built yet".
- **Never fetch on a stranger's say-so.** The channel parses updates *before* the kernel
  has ruled on identity. An incoming photo is fetched into `workspace/inbox/` so
  `see_image` has a target — but the fetch asks the same allowlist the kernel uses, and
  fails closed without it. Otherwise anyone who finds the bot can write to the disk.
- **Never let a second door be softer than the first.** `grab_frame` is `Effect.READ`
  although it writes a file, and that is deliberate: the floor judges by effect, not per
  target (`policy.decide`, step 4), so as a `WRITE` a video under `~/.secrets/` came out
  as an approvable `NEEDS_HUMAN` while the same recording via `hear` is a hard `DENY`.
  Frame capture would have been the softer way to the same content. What decides the
  effect is the target that can be *chosen* — the source, which is read. The picture's
  path is derived by the kernel (`policy.frame_output_path`), never taken from the
  arguments, so the model cannot pick where bytes from a foreign file land; the runner
  calls that same function rather than rebuilding the rule, because a rebuilt rule drifts
  and the kernel would then be judging a file that never appears.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [talos-kernel/Talos](https://github.com/talos-kernel/Talos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
