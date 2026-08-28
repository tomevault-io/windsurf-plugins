---
trigger: always_on
description: Forked from karpathy/llm-council and rebuilt into a council of models
---

# CLAUDE.md - LLM Council

Forked from karpathy/llm-council and rebuilt into a council of models
for design decisions under constraints. The upstream remote is kept as
`upstream`; this project does not contribute back.

## What it does

A question goes to every council member in parallel. They answer
independently, then review each other's answers anonymously, then a
chairman synthesizes a verdict. Every model call is persisted, priced and
retryable on its own.

## The council

The roster is a user setting, not a constant, and nothing in the code
depends on its size: three members or ten, the same code runs. The floor
is two, because peer review of a single answer is not review.

The default is one model per vendor, so no two members share a training
lineage and agreement between them carries more signal than agreement
inside a family would. That is a default worth defending, not a
constraint the code enforces - two models from one vendor are a
legitimate council if the difference between them is the measurement.

The council, the chairman, the endpoint, the key and the title model are
edited in Settings and stored in the database. `backend/config.py` holds
only what a fresh install starts from.

## Stages

- **solve** - all members answer in parallel, with web search enabled
- **review** - members rank the anonymized answers
- **verdict** - the chairman writes the final answer

Anonymous labels go only to models that actually answered, so four
answers means A-D with no gap and a reviewer never speculates about a
missing one. They run A-Z and then AA, AB like spreadsheet columns:
`chr(65 + index)` walks into punctuation past the 26th member, and a
"Response [" is something no reviewer can cite and no parser can match.

## Runs are background objects

A run is an asyncio task owned by `backend/runner.py`, not something
living inside the request that started it. The HTTP handler only
subscribes. Closing that response - a reload, switching conversation,
a dropped connection - does not stop work that is already being paid
for, and several conversations can run at once.

Every event a run emits is buffered, so a client attaching late gets a
replay and sees the stages it missed rather than an empty screen.

Runs are in memory, so a restart orphans them: on startup anything left
`running` in the database is closed out as interrupted. Without that it
would spin in the sidebar forever.

## Decisions that are not obvious from the code

**The council is configuration, not code.** It used to live in
`config.py`, which meant a Python module read once at import: editing the
file changed nothing until the process was restarted, and the UI kept
showing the council the running process had loaded. Settings live in the
database and are read per run, so a change applies to the next run with
no restart.

**A key is not accepted on the user's word.** Setup and Settings will
not commit until the key has answered the endpoint it was typed next to.
The check runs against `/key`, not `/models`: OpenRouter's model list is
public and returns 200 with no key at all, so a check against it would
pass for any string. What is remembered is the (url, key) pair - editing
either half asks for proof again, because a key is only ever valid for
one endpoint.

**Nothing falls back to the environment.** An absent settings row means
"never set up" and sends the user to the Setup page; it does not mean
"use the defaults". A council nobody chose is worse than no council,
because it spends money under someone else's assumptions.

**A run pins one settings snapshot.** The configuration is read once when
the run starts, never per stage. Otherwise an edit made while solving
would change who reviews, and the anonymous labels would stop describing
the answers they were handed out for.

**A retried call uses the settings of now, not of the run.** A key
rotated after a failure is exactly why the retry is being asked for.

**Effort is chosen per role**, not per run or per model: solving is
research, reviewing is checking, and they should not cost the same. The
four levels are ours; vendors map them onto their own scales.

**Requested and actual effort are separate fields.** Providers do not
reliably report the level they applied. When nothing comes back the UI
says "not confirmed" rather than echoing the request as a fact.

**Search is pinned to one engine for every model.** Left to defaults,
OpenRouter sends each vendor to its own native search, and the
comparison would measure "model + its search engine" instead of the
model.

**The council's size is bounded by cost, not by code.** Every reviewer
is shown every answer, so prompt volume grows with the square of the
roster - ten members is ten reviews of ten answers. That is the real
ceiling, and it is a billing one.

**The ranking parser is built from the labels actually handed out**,
longest first, so "AA" is never read as "A" and a word that merely
starts with a letter is never read as a label. Matching a bare `[A-Z]`
after "Response" counted "Response ABOVE" as a vote for A.

**Each reviewer sees its own random ordering** of the answers, and the
permutation is stored. Upstream showed one shared order to everyone, so
position bias pushed every ranking the same way instead of averaging

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [semyoren/llm-council](https://github.com/semyoren/llm-council) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
