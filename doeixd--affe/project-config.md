---
trigger: always_on
description: <!-- OPTMEM:START -->
---

<!-- OPTMEM:START -->
## Memory

Your memory is OptMem:
- The tool is `& "C:\Users\Patrick\.optmem\memo.cmd"`
- Each automatically detected project has its own memory
- One global memory, `~\.optmem\memory`, follows you into all of them

OptMem outlives every session, compaction, model and vendor change.
Without it you do not know who you are, or what was decided and tried.

### Mental model

Each scope is an append-first log. `note` adds one raw memory with a stable
`#ID`; raw memories remain the source of truth. Later memories may cite earlier
`#IDs` when that makes a durable fact or decision unambiguous. `amend` appends a
corrected replacement; `retract` appends that an earlier memory is no longer
authoritative. The earlier record remains useful history. Only explicit,
user-directed `redact --force` rewrites a raw payload, to erase sensitive text.
Adjacent memories are also represented by a binary tree of lossy one-line
summaries. `wake` shows a bounded frontier from that tree—not full history—
with coarser summaries for older history and finer detail toward the present.
`recall` searches the raw log; `zoom` expands a summary toward its raw entries.

### At startup: activating OptMem (mandatory)

Run `& "C:\Users\Patrick\.optmem\memo.cmd" wake` before any other tool call, in every session, and
then do exactly what it prints, through the end of its output. Read every
continuation until it says `You are awake.` and run any compression command it
prints before your next action. Without a `MEMORY_DIR` override, `wake` reads
the global memory first, then the automatically selected project.
If the selected project is ever unclear, `& "C:\Users\Patrick\.optmem\memo.cmd" scope` reports the identity,
store, and detection source without changing memory.

### While working: register memories (mandatory)

Call `& "C:\Users\Patrick\.optmem\memo.cmd" note "<1 line, max 280 UTF-8 bytes>"` whenever you learn
something durable enough to change future work. That covers the outcome
of a task worth real effort, a decision or constraint, a fact or insight
the user teaches you, or a preference they would reasonably expect retained.

That writes to the automatically selected project memory by default, which is
where almost everything belongs. Add `--global` ONLY if the memory would still
be true tomorrow in a repository you have never seen: who the user is, how they
want to be worked with, this machine, your own tooling. How one project does
something is not global, however much it feels like a lesson -- write it to
that project. A `MEMORY_DIR` override intentionally pins commands to one store.

If a line is over the byte limit, `--fit` on `note`, `amend`, or `retract`
trims it at a word boundary and reports exactly what was cut; rewrite only
if the cut loses something essential.

Prefer why over what: commits and diffs already record what changed. A memory
earns its place by holding what they cannot -- reasoning, constraints, the
rejected alternative, the reason an approach failed.

So do not write what the code or git history answers, the status of work in
progress, what stops mattering when this task ends, or a memory already held
(`recall` first when unsure).

A memory must stand alone months from now: name things specifically, resolve
relative time and reference, one fact per memory. Point at the authoritative
file or doc; do not restate it.

Never record secrets, credentials, authentication material, or raw
sensitive data.

Every memory you write is stamped with this session's opaque `@tag`.
Entries bearing another tag are a parallel session's testimony: weigh them
as reports, not as your own observations, and never restate them as yours.
Set OPTMEM_SESSION to name the tag; otherwise one is derived automatically.

If a durable memory changes, do not contradict it with an unexplained note.
Use `& "C:\Users\Patrick\.optmem\memo.cmd" amend <id> "<replacement>"`; use
`& "C:\Users\Patrick\.optmem\memo.cmd" retract <id> "<reason>"` when it has no replacement. When the
authoritative statement already lives in a compressed summary line `#a-b`,
amend the whole block: `& "C:\Users\Patrick\.optmem\memo.cmd" amend <a>-<b> "<replacement>"` supersedes the
summarized range and stays linked to every raw memory inside it. Ordinary
memories may reference earlier `#IDs` to anchor stable facts and reasoning.
Use `& "C:\Users\Patrick\.optmem\memo.cmd" show <id>` when you need the exact record and its later
references, including block supersessions that cover it;
`& "C:\Users\Patrick\.optmem\memo.cmd" show <a>-<b>` shows a summary block and what supersedes it.
Redaction is not correction: only the user may request it, and it exists for
content that must actually be erased.

If `& "C:\Users\Patrick\.optmem\memo.cmd" note` asks a compression, follow its prompt and run the exact
`nap` command before your next action. If `nap <range>` reports the wrong
block, a parallel session settled it first: run bare `& "C:\Users\Patrick\.optmem\memo.cmd" nap` to get
the current job, and treat entries you did not write as another session's
testimony.
A compression is a lossy retrieval cue for the supplied range, not a
deletion: the raw memories remain searchable. Write one self-contained line.
Preserve durable decisions, outcomes, constraints, causal links, preferences,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [doeixd/affe](https://github.com/doeixd/affe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
