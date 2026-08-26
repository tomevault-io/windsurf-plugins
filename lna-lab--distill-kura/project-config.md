---
trigger: always_on
description: Instructions for an agent (or a person) changing this codebase. Hosts that read
---

# AGENTS.md — working on distill-kura

Instructions for an agent (or a person) changing this codebase. Hosts that read
`AGENTS.md` — DeepSeek Harness via `dsh-agent-instructions`, Claude Code, others — pick
this up automatically.

## What this project is

A long-term memory for agents: recall by meaning, writing gated by evidence, several
independent stores so an agent mode change is a memory change. Standard library only,
Python ≥ 3.11. No dependencies is a feature — it lets the whole thing be dropped next
to any host, and it keeps the trust surface small for something that decides what an
agent believes.

## Boundaries

`docs/TRUST.md` states what a store boundary is and is not, and the honesty of that
statement is load-bearing. Two rules follow from it and must not be weakened:

**Every lookup resolves INTO `slug_set()`.** Never build a path from a caller-supplied
name and check whether it exists — that was the hole: `GET /memory/..%2Fprivate%2Fsecret`
returned another store's memory. Containment is membership in a set, not a blocklist of
characters. `contained()` (realpath + commonpath) sits behind it as defence in depth.

**Explicit reads are exact; only a MODEL's pick is fuzzy.** `read_exact()` for a slug from
a person, a `kura_read` call, or an HTTP route. Fuzzy `resolve()` stays for thinker picks
and `[[links]]`, where every candidate comes from the slug set — a deliberate deviation
from "links exact only", because in-store fuzzy resolution demonstrably connects real
links (`[[brain-memory]]` → `_study/brain-memory`) and cannot leave the store.

**Every path that writes into a store asks the policy.** Not just `remember_direct` and
`pour_verified` — an adversarial pass found `tidy()`, `Loom.persist()` and `init_files()`
writing into a frozen store, and `kura weave --no-model` destroying a memory's body via
`cloth_path`. When you add a code path that touches a store directory, the question to
answer in review is not "is this a memory?" but "would this run on a frozen store?".

Do not add a permission layer inside the process. An OS user boundary is stronger, older
and easier to verify than any token check this core could carry, and pretending otherwise
would invite people to rely on it.

## The one rule that outranks the others

**No model output becomes a stored fact without a mechanical check.**

Everything in `distill_kura/distill/gate.py` is deterministic Python, and it stays that
way. If you are tempted to "let the model decide" something the gate currently decides,
you are re-opening the failure this project exists to close: an agent asserts something,
the distiller records the assertion, the next agent reads it back as ground truth. That
loop is self-reinforcing. Prompt instructions do not stop it — that was measured, not
assumed.

Prompts may *help a model pass the gate honestly*. They may not *replace* it.

The same rule has a second face in `weave.py`: **compression may shorten a description,
but it may never lose, reorder or invent a link.** That is checked mechanically on every
weave and raises `WeaveError` if violated. Do not downgrade it to a warning. A memory
missing from the map does not exist as far as the agent is concerned, and the loss is
invisible — the cloth looks perfectly healthy.

## Things that must not grow here

**No number says how much a memory matters.** No `importance`, `salience`, `priority`
or `score` field; no `recurrence_count`; no retention decision that reads
`read_counts()`; no table of points per tag. Tags are words about a memory's
character, and the three sentences (`belongs_because` / `keep` / `may_fade`) are
curation judgements against the store's charter. The charter ranks; nothing else
does. `tests/test_rooms.py` greps the source for the forbidden names — keep it that
way rather than finding a synonym.

**A memory never changes store.** No move, no copy, no re-filing by tag, no router
that reads a message and picks a room. The store is chosen before the conversation by
the host, and a mode change affects only future sessions. If you find yourself
wanting to deduplicate across stores, stop: Research's "what we learned" and
Develop's "what we did" are two facts.

**A claiming tag needs its evidence.** `verify_tags()` in the gate is deterministic,
like the rest of the gate. `entrusted`, `emotion-carried`, `recurred`, `landmine`,
`formative` each name the evidence that would make them true; a model proposes, the
evidence decides, and both the basis and every refusal are in the manifest.
`recurred` is written once by the distiller against a prior memory from a different
journal — it is decided, never proposed, never counted.

**Forgetting is not designed yet, and this codebase must not design it by default.**
`doctor` observes capacity in four units with `limit` and `pressure` left `None`.
Anything that would pick a unit, set a limit, choose candidates, garage, settle,
absorb, release or delete is a conversation with the people whose memories they are
— `docs/DESIGN.md` §8 lists what is undecided. If a change needs one of those
answers to compile, it is the wrong change for now. The first forgetting pass will be
a dry run that modifies nothing.

## Layout

```
distill_kura/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lna-lab/distill-kura](https://github.com/lna-lab/distill-kura) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-24 -->
