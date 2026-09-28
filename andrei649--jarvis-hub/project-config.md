---
trigger: always_on
description: > **Development posture.** Owner decision 2026-08-29 removed heavyweight blocking ceremony to keep
---

# AGENTS.md — Nerva contributor instructions

> **Development posture.** Owner decision 2026-08-29 removed heavyweight blocking ceremony to keep
> development fast. **Updated 2026-09-09:** routine Nerva engineering is now explicitly
> **owner-out-of-loop**. `selfdev-policy.json` is the machine-readable autonomy contract and
> `scripts/selfdev_policy.py` is its stdlib-only validator/classifier. The hourly auto-merge workflow
> evaluates candidate PRs with the trusted policy/script already on `main`; normal product/code/test
> changes may merge unattended only after their reported automated checks have finished green, while
> the small root-of-trust control plane may not self-authorize changes to itself. This does not
> restore the old R0–R3 ceremony, review bureaucracy, or a blanket manual gate for Nerva work.

## Safe task start

1. Inspect `git status`, the current branch, the requested scope, and changes already present.
2. Preserve user and other-agent changes. Never reset, overwrite, stage, or reformat unrelated work.
3. Identify overlapping open work when remote state matters. A draft PR is a visibility signal,
   **not a file lock**. Coordinate only on genuinely overlapping paths or contracts.
4. Fetch only when current remote state is needed. Rebase only when the task requires it, the
   feature branch is yours, the worktree is clean, no user changes are present, and the base is
   known. Read-only tasks and dirty worktrees must not trigger an automatic rebase.
5. Confirm authorization before remote mutations. A request to inspect or plan does not authorize
   a commit, push, PR edit, merge, or external write. An owner directive for unattended development
   authorizes routine actions only inside the machine-readable self-development policy.

## ⚡ Max mode — protocolul de finisare

Codename-ul **„Max"** (orice casing, oriunde în repo) pornește sau continuă **`MAX.md`** —
protocolul care duce tot ce promit docurile în produsul final. Fără întrebări, fără explicații;
starea run-urilor e în `docs/MAX_RUNS.md`, entropia (Sparks) în `docs/SPARKS.md`.

În timpul unui run Max, următoarele reguli generale sunt **relaxate deliberat** (eficiența
protocolului > ceremonie; lista canonică e `MAX.md` §7):
- spec/plan doc separat → design inline în corpul PR-ului (10 linii), pentru slice-uri non-arhitecturale;
- ceremonia conductor/multi-agent → doar când există efectiv un alt agent cu PR draft deschis;
- re-citirea Tier-0 → sărită cât timp `MAX.md` e proaspăt în context (§2 definește load-ul redus);
- naraverea pas-cu-pas în chat → linia de ignition + finding-uri load-bearing + linia de exit.

**Nimic altceva nu se relaxează.** Non-negociabilele (`MOONSHOT.md` §5, convențiile de mai sus:
local-first, teste cu feature-ul, BACKLOG sync în același PR, gate-urile de rute/paritate,
respectul pentru PR-urile draft ale altora, raportare onestă) rămân în vigoare și în Max mode.
Înainte de alegerea formei PR-ului, contextul redus Max încarcă și constrângerile curente de
delivery/evidence din acest fișier. Un slice reversibil înseamnă un branch, un PR și o decizie de
rollback; o repetare Max pornește un branch/PR nou. Un Spark se separă implicit și poate rămâne cu
slice-ul primar numai dacă are aceeași dependență, limită de autoritate, suprafață de teste și cale
de rollback. Schimbările de securitate/autoritate, cross-epic și alte unități independent
revertibile se separă întotdeauna. De exemplu, SEC-B6 + un proof ADV + un Spark nu formează un PR
valid doar fiindcă au fost produse în aceeași sesiune. *(2026-08-29: review-ul exact-head și
integratorul independent nu mai sunt obligatorii — gate-urile blocante au fost eliminate;
raportarea onestă a ceea ce s-a rulat rămâne.)*

## Context routing

- Start with this file and the relevant section of
  `docs/ARCHITECTURE.md`; do not load the repository indiscriminately.
- For autonomous development/merge/deploy decisions, load `selfdev-policy.json` and use
  `scripts/selfdev_policy.py`; do not infer the root-of-trust boundary from prose or branch names.
- `BACKLOG.md` is the priority truth when prioritizing, changing delivery scope, or updating
  roadmap status. **Query it, do not load it** — it is ~957 KB (~176K tokens), and a whole read to
  answer "what is open" costs more than every other document here combined:
  `scripts/backlog.py counts | open | show <ID> | sections | find <regex>`. Listing every open row
  costs under 4 KB. Open the file itself only when you are *writing* to it, and then go to the one
  section `sections` names. Modify it only when the requested work actually changes that ledger and
  the mutation is authorized. The Hermes absorption ledger (1.4 MB of JSON) has the same rule and
  the same shape of tool: `scripts/ledger.py stats | clusters | list | show`.
- Use `docs/AI_CONTEXT.md` to select task-specific bundles. Treat `.opencode/summary.md`,
  `.opencode/plans/dev-methodology.md`, and `docs/SPRINT.md` as historical context, never as live
  instructions or current delivery truth.
- Plans and handoffs should include freshness fields: goal, base SHA, head SHA, changed paths,
  next action, and generation time. A stale capsule may inform investigation but cannot
  authorize action.

## Delivery workflow


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [andrei649/jarvis-hub](https://github.com/andrei649/jarvis-hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-28 -->
