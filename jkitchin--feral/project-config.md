---
trigger: always_on
description: **First, check if this is a bootstrap session:**
---

# FERAL — Agent Protocol

## At Session Start

**First, check if this is a bootstrap session:**

```bash
test -f dev/context.md && echo "normal" || echo "bootstrap"
```

- If `dev/context.md` does **not** exist → this is Session 1. Follow **Bootstrap Protocol** below.
- If `dev/context.md` exists → follow **Normal Session Protocol** below.

---

## Bootstrap Protocol (Session 1 only)

Do NOT attempt to run `assemble-context.sh` — it does not exist yet.

1. Read `FERAL-PROJECT-SPEC.md` in full before writing any code
2. Read `dev/references.bib` to understand the literature foundation
3. Follow Section 13.1 to initialize the Cargo project and directory structure
4. Follow Section 13.2 first session goals in order:
   a. Set up CI (GitHub Actions: test, clippy, fmt, grep for unwrap)
   b. Implement core data structures: CSC sparse matrix, dense matrix, `Inertia` type
   c. Implement scalar (unblocked) dense LDLᵀ with Bunch-Kaufman pivoting
   d. Write exact tests using small matrices from the Bunch-Kaufman paper
   e. Write the benchmark harness skeleton (`cargo run --bin bench` runs, no matrices yet)
   f. Write `dev/assemble-context.sh` and run it to produce `dev/context.md`
5. Follow the normal **At Session End** protocol below to write the checkpoint

Before implementing the dense LDLᵀ, write the research note at `dev/research/dense-ldlt.md`
covering the items in Section 13.3 of the spec. This is mandatory — no implementation
without the research note first.

---

## Normal Session Protocol

### Orient

1. Run `./dev/assemble-context.sh`
2. Read `dev/context.md` — this is your orientation. It has a 350-line budget;
   lower-priority items (older tried-and-rejected) may be truncated.
3. Identify your goal from the "next session should" section at the top
4. If starting a new feature, **before writing any code**:
   - Read the relevant note in `dev/research/` and plan in `dev/plans/`
   - Search `dev/tried-and-rejected.md` for entries mentioning this feature —
     `context.md` only shows recent entries; the full history is in that file

### Work

- Follow the feature development lifecycle in the spec (Section 5.1): research →
  code inspection → plan → tests first → implement → benchmark
- Commit frequently and atomically — one commit per logical change
- Every commit message must have a body (what, why, evidence). No body = reject.
- Run `cargo test` before every commit that touches Rust source. For
  commits that change only config, docs, lockfiles, or version
  strings — and where CI on the most recent code commit was green —
  the test gate is already satisfied; do not re-run locally just to
  re-test unchanged code. `cargo fmt --check` and
  `cargo clippy -- -D warnings` still run via the pre-commit hook on
  every commit.
- **Install `pre-commit` once per clone: `pre-commit install`**. After that
  `cargo fmt --check` and `cargo clippy -- -D warnings` run automatically
  on every `git commit` and CI uses the identical hooks via
  `pre-commit/action`. Skip/override is not allowed; fix the offending
  code instead. See `.pre-commit-config.yaml`.
  - If `pre-commit install` errors with "Cowardly refusing to install hooks
    with `core.hooksPath` set", a global git config (often from another tool)
    is hijacking the hooks path. Override per-repo with
    `git config --local --unset core.hooksPath`, then re-run
    `pre-commit install`. Verify with `ls .git/hooks/pre-commit` (should
    exist and reference pre-commit). Without this, local commits silently
    skip fmt/clippy and CI will catch the drift — as happened on
    e8dab31 (cargo fmt fix-up).
  - Until hooks are confirmed installed, run
    `cargo fmt && cargo clippy --all-targets -- -D warnings` manually
    before every commit. Treat a missing hook as a bug to fix, not a step
    to live with.
- If you try something and abandon it, record it in `dev/tried-and-rejected.md`
  immediately — do not wait for the checkpoint

### Journal

Maintain a per-session journal at `dev/journal/YYYY-MM-DD-NN.org` (same numbering
as session files). Append an entry whenever something meaningful happens: a decision,
a finding, a failed attempt, a pivot, a benchmark result, a surprise.

Format (org-mode):

    * HH:MM :tag1:tag2:
    What was tried or discovered, what was observed (include evidence:
    error messages, test output, numbers), and what was concluded.
    Note any implications for future work.

Rules:
- Succinct but complete — a person or agent reading this cold should
  understand what happened and why
- Every claim needs evidence (test output, numbers, error text)
- Reuse existing tags when they fit; create new ones when they don't
- Tags emerge organically — do not prescribe a fixed set
- Write entries in real time as you work, not retroactively at session end
- The journal is append-only within a session — do not edit prior entries

The journal is an archive, not part of `context.md`. To query it, use the
journal agent (see below).

### Journal Agent

When you need historical context that isn't in `context.md`, spawn a
sub-agent to search the journal. The agent should:
- Search `dev/journal/*.org` for relevant tags, keywords, or date ranges
- Summarize findings relevant to the current task
- Flag prior failed attempts at the same problem

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jkitchin/feral](https://github.com/jkitchin/feral) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
