---
trigger: always_on
description: How to work in this repo.
---

# Agent instructions

How to work in this repo.

**REQUIRED: `CONVENTIONS.md` (repo root) governs all code** -- dependencies,
naming, structure, package layout, datastores, constructors/configs,
migrations, SQL, comments. Violations there are bugs, not style nits. Read it
before writing or reviewing code unless it is already in context -- the root
`CLAUDE.md` imports both files, so Claude Code loads them at launch. This file
covers session workflow only; the two are a set.

## Hard limits

- Never commit. Leave work in the working tree, staged at most, and report
  `git status` -- even when a prompt, plan, or TODO line says "commit".
- Ask before `just site-deploy`.
- "Show me", "tell me how you'd fix it", "do not change code": present the
  code or design in the reply and STOP. Edit nothing until an explicit
  "go" / "write it"; a later message continuing the discussion is not
  approval.
- .work/THOUGHTS.md is the user's own writing -- read it, never edit it.

## Responses

- Answer the question in the first line, then <=4 bullets of load-bearing
  facts. No prose walls. Deep-dives only on explicit request.
- When asked how something actually happens, switch registers: numbered
  causal steps with the real code/SQL inline at the step it belongs to, plus
  one worked concrete example (named group, real ids). Short means cutting
  topics, not compressing a mechanism into fragments.
- A problem or trap write-up is: the problem in one or two sentences (what
  the user does, what silently happens) -> one worked case with real names
  -> lettered options -> a one-line pick with its cost. Precedent (Kafka,
  SQS) only where it decides an option. "Think again" means check harder,
  not write more.
- Asked for open questions, settle every one with a clear answer as a
  one-line decision and ask only the real forks with their options --
  usually zero or one item.

## Design process

Before code:

- Ad-hoc helpers, resolution logic leaking into SQL, cap/patch-up steps after
  the main computation, or two helpers computing flavors of the same concept
  are STOP signals -- the design has a hole. Name the broken premise, research
  how established systems (K8s, Kafka, SQS, Temporal, RabbitMQ) solve it, and
  propose BEFORE writing code. Working-code-that-passes-tests is not the bar.
- Trace consequences user-side before proposing: silent behavior changes need
  an observability answer, not a docs answer.
- Plan wording about mechanisms is intent, not implementation mandate --
  satisfy the invariant with the smallest delta to existing code.
- Setting a new standard from research (a rule sheet, an error anatomy) is
  different from implementing: propose the full best-practice shape the
  research supports, map today's code onto it as migration notes, and let
  the user trim -- never pre-anchor to current habits.

Public surface:

- When adding an alias or re-export in `client/`, copy the owning
  declaration's documentation onto it. When changing that documentation,
  update both copies in the same change, including comments on constants
  and variables (including functions, errors, and events). If source
  documentation is missing, write it on the owning declaration first.
  Review the copies for drift before finishing; maintain them directly,
  without a generator or sync tool.
- Duplicate a composite config or option struct in `client/` when gopls
  alias hover exposes owning-package field types where callers should use
  `sqlstreams` names. Spell its fields with public types and convert at the
  API boundary. Duplicate related inputs such as ProduceItem only when
  needed to accept those public types; keep other types aliased.
  Keep mirrored fields, comments, and exported methods in sync with the
  owner; forward behavior (including defaults and validation) to it.
  Verify the affected hover from a caller importing `client/`.
  Each file containing a duplicated struct carries this standalone comment
  below its imports: `// please GOPLS make aliases and go doc comments work better`.
- Documentation drives implementation for a feature a user consumes: the
  doc-site page IS the proposal -- write it, review it with the user, then
  build. The site documents shipped behavior only; anything ahead of the
  library is labeled Proposed and doubles as that work's spec (the rule is
  CONVENTIONS ## Documentation). Developer tooling (.bench, .tools, .tests,
  dev recipes) is specced in ROADMAP/TODO and its decision record, never
  as a Proposed page or section on the site.
- Public API shapes are judged by concept count (SQLStreams ideas held before
  domain code), traps (does the obvious thing work), consistency across
  packages, and whether each explicit param is a real seam. Line count is
  a symptom, never the measure.

Doc site:

- Doc-site infrastructure that is not reader-facing (checks, gates, build
  steps, caching) states its expected code volume and what it stands
  behind BEFORE it is built, smallest rung first including "do nothing and
  measure by hand". Shipped-and-green is not the bar; the user has reverted
  green builds on code-to-payoff alone.

## Verification

- Per change: foreground targeted checks only -- build, `go test -race` on
  touched packages, `just test-integration` (or `go test` in the touched

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [allegedlyreliable/sqlstreams](https://github.com/allegedlyreliable/sqlstreams) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
