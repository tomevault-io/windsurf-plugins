---
trigger: always_on
description: The working contract for Claude Code, Codex and their subagents in this repository.
---

# AGENTS.md

The working contract for Claude Code, Codex and their subagents in this repository.
`CLAUDE.md` symlinks here, so both read the same rules. This file holds principles and standing
operator decisions. The operator's plan and Git history own the forward queue, the current head,
run condition and open decisions. Newer evidence wins wherever they disagree, and a rule whose mechanism has left the source is dead text — say so
rather than obeying it.

Be eager: given ambiguity, do the task rather than ask. Carry authorised work through
implementation, checks and delivery; authorisation persists across turns and compaction. Ask only
for a decision or permission that is genuinely missing, with the reviewable result ready and the
blocking rule cited.

<!-- weekly-best-run:start -->
<!-- weekly-best-run:end -->

**Bun only.** Stable Bun 1.4.2 from `.bun-version`, for every repository task including one-off
inspection: never Node, npm, npx,
compatibility prefixes, version managers or package-manager handshake-variable changes.
A same-version canary is a different runtime. If pinned Bun is unavailable,
report the environment gap. Running runs keep their opening's executable bytes until they finish.
Exception: for Astra, prefer Python for ad hoc inspection, data handling and automation scripts;
repository source and its test and gate commands stay on TypeScript/Bun.

## What Anabasis does

One short prompt about a technical domain becomes two products: an agent harness that solves tasks
in that domain, and an evaluation that decides whether those solutions are correct. The evaluation
is the harder product. Plausible tasks are cheap; a correctness model that accepts correct answers
and rejects convincing wrong ones is what makes a measured pass rate mean anything. A rule matters
only when construction carries it into the product or a check enforces it; a check that never
changes a decision is decoration.

```text
INPUT      one-line prompt (+ optional public --context paths)
   │
BUILD      Builder session (model)  ── reads STARTER.md, maps the field into task families,
   │       installs real tools under .toolchain, writes correctness-model/ and agent/,
   │       rehearses with harness_inspect → harness_trial → correctness_check, then submit
   │
GATE       submit (code)  ── immutable snapshot → bundle contract → validation → conformance
   │       probes → control census → F2 reference solve of every task → grounding/solvability →
   │       adopt or refuse; every refusal returns typed findings to the same session
   │
MEASURE    Built Harness (model)  ── one confined solve per task with the closed tool roster;
   │       each accepted artifact goes to the host verifier (code plus hashed installed tools)
   │       → verified | unaccepted | non-result
   │
LEARN      review slot (model, advisory)  ── Main Judge battery review, diagnosis reader,
   │       Epoch Reviewer, deterministic rebuild advice packet
   │
NEXT MOVE  boundary (code)  ── build | measure | rebuild | stop
           The Builder chooses and implements the next experiment; accepted bytes own attribution
```

1. **Input.** One line names a field, not a deliverable ("designs steel roof trusses to
   Eurocode 3"). No plan, answer key, driver or hint is added; public `--context` files are the
   only extra input.
2. **Build.** The Builder reads `STARTER.md` in a seeded workspace and writes the whole bundle:
   `correctness-model/brief.json` (domain plan and public rule decisions), `.../tasks.json`
   (public inputs plus hidden expectations), `.../controls.json` (task-bound accept and reject
   artifacts), `.../evaluator.ts` (named Boolean checks), `.../reference/index.ts` (a public-input
   reference solve), `agent/tools-spec.json`, `agent/tools.ts`, `agent/BUILT_AGENTS.md` and
   `agent/config.yaml`. It installs the domain's real open-source tools before writing around them.
3. **Gate.** `submit` freezes the candidate once and runs one sequence on that snapshot alone.
   A refusal keeps the session; acceptance ends it.
4. **Measure.** The Built Harness solves the battery under its own file wall with the tools the
   Builder wrote. The host verifier runs the declared checks over each accepted artifact, with
   every tool run hashed and recorded as an evidence row.
5. **Learn.** The review slot reads recorded rows and traces and writes advice. It never changes a
   pass, an acceptance or a claim.
6. **Next move.** Code admits construction, measurement, an adopted-product continuation or a
   typed stop. The existing Builder chooses the next experiment from recorded evidence; there is no
   separate planner. Its bounded `EXPERIMENT.json` records the gap, change, expected result, scope
   and a `target` of `{comparator: "at-least" | "at-most", verifiedPasses}`.

### Design priors

These ten decisions are settled. Code that quietly moves one of them is a defect, not a choice.

1. **Correctness has one owner.** The host verifier, running the declared checks and installed
   tools, decides every pass. No model judge, review or Builder claim sets a score.
2. **The input is one line.** No hidden plan, custom driver or evaluator hint rescues a launch.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [s-smits/anabasis](https://github.com/s-smits/anabasis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
