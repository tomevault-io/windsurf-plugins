---
trigger: always_on
description: Conventions for anyone, human or agent, changing this repository.
---

# Working on OpenScript

Conventions for anyone, human or agent, changing this repository.

---

## The rules that are not negotiable

Twenty seven checks enforce these, twenty six `check-*` scripts and the harvest
run with `--check`, and each one of them is a rule somebody broke once.
`npm test` runs all of them, and so does every pull request. The count is here
to be corrected when it changes, not to be trusted: `package.json`'s `test`
script is the list.

1. **No `eval`, no `Function` constructor, no dynamic code construction.** The
   compiler emits data. This is what lets the language run inside an application
   with a strict content security policy, and it is the reason a platform can run
   many customers' scripts in one process. It is not a preference.
2. **Zero runtime dependencies.** `dependencies` is empty and stays empty. Every
   dependency is something an adopting platform has to accept.
3. **Layering.** A module is a directory and its index is its only door. Nothing
   under `src/core` may import a package or touch a browser global. An adapter is
   the only place allowed to know two worlds at once.
4. **No code file over 500 lines**, wherever it sits: source, tooling or test. A
   file past it is usually two things that were never separated. A document is
   prose and is not counted. One already over it is recorded in
   `spec/modularity-exceptions.json` with the length it was recorded at, which is
   a ceiling it cannot grow past and a row that has to go the moment it is
   earned out.
5. **Name nobody.** No outside product, platform, company, trademark, market index
   or real instrument, anywhere: not in source, comments, documentation, examples,
   test names or commit messages. Describe prior art generically. Examples use
   placeholder symbols.

   **What the check covers is narrower than the rule, and a green build is not
   proof of the rule.** `scripts/check-names.mjs` reads every file in the tree
   against a fixed list: about eighteen products and platforms, and thirteen
   indices and instruments, each stored encoded so that the checker is not the
   one file breaking the rule it enforces. A name on that list is caught
   anywhere, in any file; a name that is not on it passes, and so does every
   name nobody has thought of yet. The list also leaves out, on purpose,
   identifiers that are ordinary English words, because a build that fails on a
   sentence costs more than the leak it would catch.

   So the mechanical half is "these names, everywhere", and the rest is
   attention: a reviewer reading a new comment, a new example symbol or a new
   commit message. That is the whole of what can be mechanised here, and it is
   written down rather than implied, because this repository's own standard is
   that a check overstating its reach is worse than a small one stated
   truthfully. A name that does get through is added to the list in the same
   change that removes it, so the list grows by the cases attention missed.
6. **No fact stated twice.** A value set written out in two files is a fact you
   would have to edit two places to change, and every copy reads as authoritative.

Plain text everywhere: no emoji, and no em dashes or en dashes. Use a comma, a
colon, parentheses or a full stop.

## Diagnostics

Every error carries a catalogue code. Never throw a bare string.

**The message says what is wrong and the fix says what to do, and both must be
true of the actual program.** A fix that would change the meaning of the code is
worse than no fix, because a reader trusts it. An error that cannot suggest a
concrete fix is a badly designed error; redesign the error.

Error codes and their text live in `spec/errors.json` and are generated into the
compiler. Nothing under `src/` retypes a message.

**A documented refusal is a refusal something raises.** A code in the catalogue
is raised by some code path, or the entry carries a `deferred` sentence saying
what happens instead today and what has to exist first. There is no third case:
a code taught as current behaviour that nothing can produce is a promise nothing
keeps, and `scripts/check-raises.mjs` fails the build on one. The deferral goes
in the catalogue, never in the checker, because a list of exemptions inside a
check is read by nobody and grows by a line whenever somebody is in a hurry.

**A worked example is compiled, and its mistake raises its own code.** Every
entry's `after` block goes through the compiler, because it is the fix a reader
is handed at the moment they are stuck and they will paste it; every `before`
block is compiled, and run where the code needs a bar, and has to raise the code
it is filed under. `scripts/check-examples-compile.mjs` does both, and what it
cannot reach it says and counts rather than skipping. The three states it
accepts are fields on the entry, beside `deferred` and for the same reason: an
`unexercised` sentence for a code no example can reach, and an example `kind` of
`transcript` for the entries whose example is the host's input rather than
source.

**A fix sentence is held to the language this release has.** The blocks are
compiled and the sentence beside them is what a reader acts on, so every call a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [marketcalls/openscript](https://github.com/marketcalls/openscript) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
