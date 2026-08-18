---
trigger: always_on
description: Flatte (package `flatte`, module path `github.com/lunguini/flatte`) is a
---

# Flatte Development Guide

Flatte (package `flatte`, module path `github.com/lunguini/flatte`) is a
functional TUI foundation for Go: single mutable state struct, `View` as a pure
function of state, direct mutation instead of TEA message dispatch, async
funneled through named `StateUpdate`s onto a single-writer loop. It is the
deliberate inverse of Bubble Tea while reusing Charm's MIT substrate (Lip Gloss
v2 for styling; ultraviolet for input parsing and cell-buffer rendering,
wrapped behind Flatte APIs).

## Mission and quality bar (read this first)

Flatte is being built to be **the production-grade, idiomatic-Go Bubble Tea
alternative** — software teams ship on and depend on, not a research spike.
**This is not experimentation.** The phased roadmap and the "abstraction is
found, not designed" rule are about *engineering rigor* (don't add surface
ahead of evidence), **not** about hedging on whether to deliver. We are
committed to carrying the roadmap through a 0.1 release, then to a 1.0 once
community feedback and API stabilization justify it.

What that means in practice, on every increment:

- **Reliability and correctness are non-negotiable.** Treat it as if critical
  systems depended on its correctness. Every code path is tested; no path is
  left "probably fine."
- **Full QA, every time:** TDD (red→green), unit tests for logic, golden
  snapshots for views, `-race` on touched packages, `go vet` on darwin **and**
  `GOOS=windows`, `gofmt` clean, and a real-terminal (TTY) pass wherever
  behavior is terminal-conditional. Benchmark goldens stay byte-identical.
- **Honesty over advocacy stays** — the evaluation log still records what got
  worse. Honest evidence is a *quality control*, not a sign of tentativeness.
- **Keep momentum.** Drive each well-scoped unit to done and continue to the
  next; don't stop to ask "should I proceed?" between them. Stop only when
  genuinely blocked, when a decision is truly the user's to make, or at a
  required TTY/verification gate. Finishing the roadmap is the job.

## Document map (read in this order for context)

Internal docs live under `.docs/` with **opaque filenames** (`d01.md`,
`d02.md`, …) so the public repo can't infer topics from names;
`.docs/index.md` (encrypted) maps each opaque name to its role — read it first.

| Document | Role |
|---|---|
| The design doc | Thesis, principles, architecture, phased roadmap. Direction lives here. |
| The status tracker | **Living implementation tracker** — what exists, known bugs, what's next per phase. Truth lives here. |
| The evaluation log | Evidence log from dogfooding — what extracted cleanly, what's worse or unproven. Append-only in spirit. |
| Plans | Implementation plans for individual pieces of work. |
| `README.md` | Public project introduction and minimal app. |
| `quick-reference.md` | Public API map for humans and AI agents building Flatte apps. |
| `examples.md` | Public sample catalog and what each sample demonstrates. |
| `AGENTS.md` | Contributor and agent workflow rules. |

## The `.docs/` folder (internal project docs; sensitive, git-crypt)

**All internal project documents live in `.docs/`** — design, status tracker,
evaluation log, plans, and any specs or design/brainstorm notes. Files use
**opaque names** (`dNN.md`); `.docs/index.md` (encrypted) maps them. There is
no `.specs/` folder and no `.docs/plans/` subfolder; use `.docs/` exclusively.
Public user-facing docs live at the repository root. Rules:

- **Rely on it.** Before designing, implementing, or claiming anything
  about project state, read the relevant `.docs` file first (use
  `.docs/index.md` to find it). The status tracker is the authoritative answer
  to "what's implemented and what's left" — trust it over assumptions, and
  verify it against code when in doubt.
- **Update it.** Any change that affects what these documents say (new
  feature, fixed bug, design decision, new finding) updates the matching
  `.docs` file in the same commit. A new internal doc takes the next free
  opaque name (`dNN.md`) and a row in `.docs/index.md` — never put the topic
  in the filename.
- `.docs/**` is encrypted with **git-crypt** (see `.gitattributes`). All
  sensitive or internal material goes under `.docs/`, never in public files or
  code comments. (`.specs/**` stays git-crypt-encrypted purely as a safety
  guard against accidental plaintext — it is not a place to put files.)
  git-crypt encrypts file *contents*, not *names*, which is why `.docs/` uses
  opaque `dNN.md` names mapped only in the encrypted `.docs/index.md` — keep it
  that way so the public repo leaks no topics.
- Do not copy `.docs` content into public files, commit messages, or
  README-style docs.
- If `.docs/` files look like binary garbage, the repo is **locked** — stop
  and ask the user to run `git-crypt unlock`. Never overwrite an encrypted
  blob with plaintext.

## Commands

```bash
go test ./...        # full test suite, includes golden snapshots
go vet ./...
cd cmd && go run ./flat-modal        # run a sample app (needs a real TTY)
```

Golden snapshots have **no auto-update flag**: when a view change is
intentional, edit the golden file under the sample's `testdata/` by hand
(or rewrite it from the test's "actual" output) and re-run the test.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lunguini/flatte](https://github.com/lunguini/flatte) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
