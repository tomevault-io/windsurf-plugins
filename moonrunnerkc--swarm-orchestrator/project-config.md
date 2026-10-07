---
trigger: always_on
description: `swarm` is a coding agent for the terminal that leaves a signed, offline-checkable record of
---

# The coding agent (advanced beta)

`swarm` is a coding agent for the terminal that leaves a signed, offline-checkable record of
what it ran and what passed. It is the advanced mode of this repository and is in beta: the
verifier, [`swarm-verify`](../README.md), is the product, needs none of what is on this page,
and is where a new user starts. Everything below is preserved for people who run the agent.

## What it is

`swarm` takes a task and a git repository, makes a bounded change, runs your project's checks
over it, and writes a record of every command it ran and every check that passed or failed.
The record is a hash-chained ledger, exported as a signed bundle that carries its own
dependency-free verifier, so anyone can check it later with nothing but Node and without
trusting the machine that produced it.

The model can say whatever it likes. It cannot make a check pass, mark a claim verified, or
change a record after the fact.

**Upgrading from 12.x?** That was a pull-request auditor; 13 and later are a coding agent under
the same package name, with no migration path. Stay on the `v12-final` tag, or pin the major.

## Install

```sh
npm install -g swarm-orchestrator
```

It runs on Node 22 or newer, and every measurement runs on Node 22.8 or newer: the
changed-line coverage measurement spawns node's test runner with process isolation named on
the command line, in the spelling the running Node takes. On 22.0 to 22.7 that one
measurement reports unmeasured and never counts as a pass. `swarm doctor` says which Node it
found and what owns the `swarm` command, and `--fix` repairs an install that an older build is
shadowing.

## Try it

```sh
export ANTHROPIC_API_KEY=...       # or OPENAI_API_KEY, GOOGLE_GENERATIVE_AI_API_KEY,
                                   # or start Ollama and pass --model local:<id>
cd your-repository
swarm "make slugify collapse whitespace and strip punctuation"
```

Keys come from the environment or your OS keychain, never from `swarm.toml`: that file is
committed and cloned, so a key in it has already been shared with everyone holding the
repository. Every command and flag is in [cli.md](cli.md).

## How it verifies

Every tool call and every check goes through one recording chokepoint, and the record is an
append-only, hash-chained ledger that lives outside the workspace. When a run ends it is
exported as a signed bundle carrying a dependency-free verifier, so the bundle can be checked
anywhere with plain Node.

Five words carry most of the weight. A **gate** is a check declared as data: a command, a
parser, and whether it blocks. The **ratchet** is the rule that a retry may not trade away
tests, assertions or coverage to turn a gate green. A **bond** is one file a passing gate is
handed that it must refuse, so a pass that cannot fail is caught. An **oracle** is the check
you supply that says the task was done, as distinct from nothing broke. **Reach** is whether
that oracle executed the lines a patch added. And **unmeasured** is a verdict of its own:
nobody checked is not the same as checked and passed, and it never renders green.

The mechanics, the nine-answer report and worked examples are in [verifying.md](verifying.md).
Retries under the ratchet, sessions and several workers at once are in [using.md](using.md).

## Measured

Every number here links to the committed artifact of the thing happening. The full table, and
the list of things that may not be said, is [claims.md](claims.md).

- **One changed byte breaks verification.** The same bundle verified and then tampered with in
  a single byte, exit 0 and exit 1 side by side, with a script to reproduce it:
  [tamper demo](evidence/2026-08-18/tamper-demo).
- **A bundle verifies on a machine that has never seen this repository.** Run in a `node:24`
  container with no network and no mount of this repository:
  [clean-container-verification.md](evidence/2026-08-23/clean-container-verification.md).
- **A green verdict is computed by the harness, and the model cannot produce one.** In a real
  run the model asserted a predicate the language does not parse; the harness rendered it
  `UNVERIFIED` and carried on, twice: [shakedown results](evidence/2026-08-18/shakedown/results.md).
- **A suite passing is not the task being done.** Four of eighteen patches this project's own
  agent and a baseline arm produced over three public TypeScript repositories passed their
  project's whole suite and failed a hidden acceptance test, which is why `swarm ci` reports
  `regression` and `task` as two answers: [verifying.md](verifying.md). That is a measurement
  of those eighteen patches, not a rate for AI-written pull requests in general.
- **The oracle is judged too.** Certified tasks turned out to rest on oracles that could not
  fail. The last false green standing, `commander#1671`, was certified by an oracle that runs
  every line the patch adds and never tests the precedence those lines decide; it is refused now
  because that oracle accepted a change to a line it had run:
  [verifying.md](verifying.md#the-oracle-is-judged-too).
- **The September 6 mined-corpus false-green rate was 0 in 15**, 0.0%, 95% CI [0.0, 20.4], with
  two oracles per task, one handed to the tool and one held back:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [moonrunnerkc/swarm-orchestrator](https://github.com/moonrunnerkc/swarm-orchestrator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
