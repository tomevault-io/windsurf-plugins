---
trigger: always_on
description: This repository distributes agent skills. Do not treat a skill as active merely
---

# Instructions for local agents

This repository distributes agent skills. Do not treat a skill as active merely
because its files are present in context. Match the user's request through the
single portable router at `.agents/skills/promise-machine/SKILL.md`, then read
the selected plugin runtime contract and the one canonical `SKILL.md` it names.

## Collective identity

Before interpreting `Shog`, `Shoggy`, `Shoggoth`, `Big S`, `the Goth`, or
`Shoggoth Interceptor`, read the shared
[Shoggoth collective identity](SHOGGOTH.md). Those names affect how people
address the agents and skills. They do not activate a skill, grant authority,
or weaken the Promise Machine and target-repository instructions.

## Promise Machine contract

Before selecting or running a skill, read the suite-wide
[Promise Machine contract](PROMISE_MACHINE.md). Its identity is
`promise-machine/v1`. Each result authorises only the transition declared by
its canonical skill; missing, stale or insufficient evidence blocks that
dependent transition while leaving inspection, repair, rerun and safe exit
available.

The root law is authored once. Plugin-local `PROMISE_MACHINE.md` files are
generated installation copies and must remain byte-identical to it.

## Marketplace boundaries

The fourteen plugins form one marketplace, not fourteen competing descriptions
of the same job. Alexandria preserves lending inputs; Tabularium interprets
preserved venue records; Probitas assembles a counterparty dossier. Lazarus
preserves the finite historical Ethereum state and exact RPC traffic a test
needs, while Ariadne binds a released artefact digest to its evidence. Berean
holds a protocol agent's recorded answers to pinned corpora and preserved
chain reads; it neither chunks documents nor preserves chain state itself.
Pandects supplies reviewed credit laws, Hermes measures a single
gas-optimisation class named by a rule from its pinned corpus,
Hexaemeron controls a receipted delivery loop and holds each of its phases to a
named skill, while Lemma stops after producing
source-linked chunks. Horos decides what an agent does not read. Janus checks
what a contract hook may observe and change around a host action, where
Pandects supplies the economic laws such a transition must preserve. Sapheneia
shapes the agent's replies for AuDHD readers and has one bounded operation for
durable audit, issue, and comment prose. It does not change another skill's
facts or gates. Brevitas controls the volume and structure of engineering prose
after vocabulary and register passes. If a request crosses one of those
boundaries, hand it to the named sibling rather than broadening the selected
skill.

## Issue queues

Work arrives from four places and each is told apart by its issue title prefix,
so a reader knows which queue a thing came from without opening it.

- `{skill}-next`, labelled `held-job`. A ledger's held frontier job. The system
  named it; closing one increments an evolution counter.
- `{skill}-N`, labelled `wish`. The closed set from a one-off generated
  wishlist, #317 to #334. Exogenous nice-to-haves. Nothing mints another.
- `{skill}-wish`, no queue label. Something a Fiat run noticed about one skill
  and had no authority to fix inside its own packet.
- `framework-N`, labelled `observation`. Something a run noticed about the
  system as a whole. Its body opens by stating that Protasis decides which
  skill or skills it upgrades, because the filer is the wrong party to guess.

`{skill}` is the skill's own governed name rather than its plugin's, so Lemma's
is `lemma`. The reasoning, the alternatives and the two questions still open are
in [ADR-009](docs/decisions/ADR-009-four-issue-queues-and-their-titles.md).
Filing an issue merely to satisfy a workflow remains forbidden; these
conventions say how to title one that was worth filing.

Closing a delivered issue belongs to whoever merges its pull request. The
Atlas draws from open issues alone, so one whose delivery has merged keeps
being allocated until it is closed, and a contributor working from a fork
cannot close it.

## Issue and comment publication

Before an agent publishes a GitHub issue title and body or a GitHub issue
comment for this repository, use this sequence on the complete candidate:

1. freeze the required title prefix, body opening and protected evidence inventory;
2. apply `sapheneia-durable-record-shape`;
3. run Imprimatur and clear every reported defect without dropping protected content;
4. apply Vulgate to the surface only and compare its content with the source; and
5. re-run Imprimatur on the exact publishable bytes.

The four frozen title forms are `{skill}-next`, `{skill}-N`, `{skill}-wish`, and
`framework-N`. Keep every queue-specific body rule from the section above.
The protected inventory includes claims, qualifications, unknowns, negative
evidence, identifiers, paths, `file:line` locations, hashes, addresses,
selectors, numbers, dates, links, quotations, severities, verdicts, status, and
required host structure. Do not publish after a failed check, changed prefix or
body opening, missing protected item, or content mismatch.
GitHub does not enforce this repository rule; it governs agents working from
these instructions.

## Repository map


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wildcat-finance/skills](https://github.com/wildcat-finance/skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
