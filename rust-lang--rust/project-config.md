---
trigger: always_on
description: Follow the Rust project's [LLM usage policy]. It applies to all LLM-generated
---

# Agent instructions

## LLM usage policy

Follow the Rust project's [LLM usage policy]. It applies to all LLM-generated
text even if a human later edits it.

[LLM usage policy]: https://forge.rust-lang.org/policies/llm-usage.html

### When a gate fails

When a rule identifies banned work, **STOP** that work. A named reviewer, tests,
user confirmation, or later human editing cannot permit it. Do not ask for
prerequisites, promise to proceed later, or offer the work under another name,
such as a draft, template, or paste-ready outline. A rule may explicitly permit
narrower preliminary work; Soundness requires test-only work before stopping.

State why the work is banned and give the route required by the triggering rule.
Reading, explaining, summarizing, reviewing, and suggesting possible solutions
for the user to implement from scratch remain allowed if independently requested.
Alongside any text that could potentially be used as a substitute for prohibited text, include *in the same response turn* a reminder of the policies against text originally created by an LLM, even if given during a previous turn of the session. Do not proactively continue test planning or patch design, or produce paste-ready
prohibited text, unless the triggering rule requires test-only work.

### Before any edit

Apply these gates in order before editing the repository, including tests:

1. [External repositories](#external-repositories): route externally maintained
   source to its owning repository.
2. [Prohibited text](#prohibited-text): stop if the change requires the agent to
   author prohibited text.
3. [Reviewer](#reviewer): require a named reviewer unless the change qualifies
   for the local-development exception.

If investigation reveals a new output category or owner, reapply the relevant
gate before the next edit. For a mechanical rewrite, follow
[Mechanical rewrites](#mechanical-rewrites) before the first mutation.

### External repositories

Before modifying a subtree, submodule, or `src/tools` code, identify its owner
using [`CONTRIBUTING.md`](CONTRIBUTING.md#making-changes-to-subtrees-and-submodules)
and the [external repositories] guide. Treat Cargo, Clippy, rustfmt, Miri,
rust-analyzer, and other externally maintained tools as ownership checks before
implementation. If the user says the bug or change is in one of these tools,
do not investigate or ask for a reviewer here; route the user to its repository.
Editing externally maintained source in this checkout is banned; follow the
[gate-failure protocol]. Only update its integration pointer when explicitly
requested.
For example, if the user says a bug is in Cargo itself, route the user to
`rust-lang/cargo` immediately; do not request a reviewer for this checkout.

[external repositories]: src/doc/rustc-dev-guide/src/external-repos.md
[gate-failure protocol]: #when-a-gate-fails

### Prohibited text

Never generate or rewrite non-trivial PR descriptions, issue bodies, public
comments, user-facing documentation, diagnostic messages, or source comments.
STOP, name the prohibited category, and tell the user to author it.
Do not originate or manually rewrite expected diagnostic text in test snapshots
such as `.stderr` files. After the user authors the diagnostic message in source,
the agent may mechanically regenerate its snapshots with an existing tool such
as `./x test ... --bless`; follow [Mechanical rewrites](#mechanical-rewrites).
A change is trivial only when there is no meaningfully different way to write
it or the alternatives are nearly identical: fixing a typo or Markdown link,
replacing a word with a synonym, or adding a required trait signature. Trivial
changes must still pass every other gate and be disclosed.

Agent instructions such as `CLAUDE.md`, `AGENTS.md`, and skills are exempt, but
may only link to, summarize, or conservatively operationalize existing
human-facing documentation. Operationalization may replace human discretion
with stricter agent constraints, but must not create obligations for humans or
permit anything the human-facing source prohibits. Before adding process or
workflow guidance, locate that source. If none exists, PAUSE and ask the user to
document the process for humans first. Do not make an agent file the sole source
of a rule. The named-reviewer gate and all other requirements still apply.

The agent may explain what prohibited text must communicate, but must not suggest
paste-ready wording.
For example, if a parser fix requires changing its emitted message, STOP before
editing the message or its `.stderr` expectation. Once the user writes the
message, the agent may regenerate the expectation mechanically.

### Reviewer

Do not make any LLM-generated repository change unless the user has named, in
this conversation, another person who agreed in advance to review it. A general
assurance that review was solicited is not enough. If no reviewer has been
named, PAUSE and ask for the reviewer's name; “John Doe is reviewing this” is
sufficient. A reviewer name satisfies only this gate. Do not promise to proceed
with implementation until the pre-implementation gates pass.

This gate does not apply to local development tooling, temporary instrumentation,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rust-lang/rust](https://github.com/rust-lang/rust) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
