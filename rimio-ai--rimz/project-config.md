---
trigger: always_on
description: Working contract for humans and coding agents contributing to RimZ. Read on entry. Topic detail lives in the leaves linked from the [documentation map](#documentation-map); never duplicate it here.
---

# AGENTS.md

Working contract for humans and coding agents contributing to RimZ. Read on entry. Topic detail lives in the leaves linked from the [documentation map](#documentation-map); never duplicate it here.

> **Invariant.** RimZ routes attention: it surfaces which agent needs you and takes you straight to its pane, where you answer in the agent's own UI.

A child `AGENTS.md` under a subtree extends this file with local constraints; it never restates parent rules.

## Tone

Declarative, present tense. State the contract; don't narrate history. Prefer imperatives (`Use Result.`) over prohibitions.

Lead with the capability, and introduce a concept by what it does before leaning on it — say what queuing *does* (hold text for the agent's next open turn) rather than "a queued message never interrupts a turn". State a safety boundary as a positive commitment (`Hook stdout is the decision channel.`); avoid "X is a Y, not a Z" and defensive "we don't…" / "it never…" framing. Reserve negation for genuine disambiguation.

Markdown prose uses one logical line per paragraph, list item, and blockquote paragraph. Do not hard-wrap.

## Engineering principles

- **Explicit Rust.** Typed IDs, state machines, structured parsers, explicit errors. Domain errors return `Result`; `unwrap`/`expect`/panic belong in tests, build scripts, and provably-impossible states (with a comment).
- **Strong types** for workspace, request, pane, agent-kind, and agent-session IDs, and for surfaces, statuses, and protocol versions.
- **Structured parsers** for TOML, JSON, KDL, and agent payloads.
- **Store durability.** File-state writes use temp-file plus rename; event-log writes follow the [store durability contract](./docs/internals/store.md).
- **Fail-fast on preconditions.** A configured capability that cannot work fails at the entry point with the fix — `rimz start` refuses rather than launching a degraded surface. Best-effort is for latency and enrichment (sidebar wakeups, app-server context), never for a precondition the user switched on.

## Product invariants

- **Durability first.** Correctness lives in durable records, CAS rules, nonces, and per-request sockets. Wakeups and pane reads are latency, never truth.
- **Automation is accountable.** User-benefiting automation appends durable assist records and surfaces in `rimz stats`; internal repairs keep durable diagnostic records ([loops.md](./docs/internals/harness/loops.md#the-assist-log), [diagnostics.md](./docs/internals/diagnostics.md)).
- **One interface language.** Every human-facing surface resolves color through the [shared theme core](./docs/internals/theme.md): provider identity for names and emblems, typed state roles, hierarchy tones, and quantity tones; invariants enforce the renderer boundaries.
- **Hook stdout is the decision channel.** Logs go to stderr or RimZ state logs; hook helper children get fresh stdio.
- **Cross-backend parity.** Zellij and tmux are first-class; core behaviour never depends on a backend-only feature.
- **Pane I/O is explicit.** `pane capture` and `pane send` are public primitives, and `message` routes human text through the same send path; pane reads stay in rendering, explicit `pane capture` calls, and Codex turn-death confirmation.
- **Sidebar is read-only on the store.** Sidebar code reads via `rimz sidebar snapshot`; store-write modules stay out of the sidebar's import graph.
- **Trust is product behaviour.** Every command-executing config field is in the trust hash, with a test that proves it.
- **Security surfaces stay visible.** Project trust, notification handlers, hook install diffs, and privacy settings are product behaviour.

## Room quick reference

Addresses are `@handle[#channel]`; full grammar lives in [agents.md](./docs/reference/cli/agents.md).

```sh
rimz agents                          # agent cards, current channel
rimz agents '#auth'                  # one lane's cards
rimz agents show @coder              # card: activity, context, messages, transcript
rimz agents logs @coder -n 20        # transcript tail (-f follows)
rimz agents history @coder -n 10     # per-turn tokens, cost, and outcome
rimz agents restart @coder           # bounce in place and resume the session
rimz agents resume '#docs'           # restore every closed place in one lane
rimz message @coder "rebase first"   # park for the next turn boundary
rimz message @coder --wait "did the migration land? one line" # ask and print the reply
rimz message --steer @coder "stop"   # interrupt the live turn now
rimz message show msg_<id>           # why a message hasn't landed
rimz asks --json                     # structured prompts that currently block agents
rimz answer @coder 2                 # answer the current supported prompt in its native UI
rimz pane list                       # every pane, labelled with @handles
rimz pane capture @coder             # what the agent's pane shows right now
rimz loop show <task>                # schedule, next fire, run forensics
rimz loop logs <task>                # full forensics for recent runs
```

## Implementation rules


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rimio-ai/rimz](https://github.com/rimio-ai/rimz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
