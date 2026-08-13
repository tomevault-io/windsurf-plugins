---
trigger: always_on
description: **Read this file in full before doing anything.** It is the canonical instruction set for this
---

# HunterStick — operating instructions

**Read this file in full before doing anything.** It is the canonical instruction set for this
repository. `CLAUDE.md` and the other runtime files are pointers to it.

---

## Session start

Run this first. It is the same entry on every runtime:

```
tools/brief.py
```

It prints a compact snapshot of the named targets, their manifest state, and blockers. Add
`--rules` prints **this file verbatim** — there is no summarised second copy to drift. It reads the target files rather than maintaining another
state store. `index.md` holds decisions and state, not counts — ask `kb.py runs` for the
ledger and `coverage.py` for what is untested. Paste
the `--rules` output as your first message on a runtime that does not auto-load this file — or
just tell the model to read `AGENTS.md`, which is better whenever it can reach the filesystem.

Then read, in this order, only what applies:

| # | file | why |
|---|---|---|
| 0 | `tools/config.py check` | machine config — researcher identity, VPS, API keys. **Never read `config.json` directly**; it holds secrets. `check` redacts credentials. Unset values are named blockers, not silent defaults. |
| 1 | `targets/TARGETS.md` when present | the local, gitignored registry. `tools/brief.py` also discovers target directories directly. **There is no "active" target.** |
| 2 | `README.md` · `TOOLS.md` | **only when explaining or auditing HunterStick itself**, not during an ordinary hunt. GitHub overview · tool reference. |
| 2a | `playbooks/system/orchestrator.md` | **always load, first.** **Who you are** — a security researcher, your edge, what you are accountable for — then the loop, the delegation rules, and the boundary you do not cross. |
| 3 | `playbooks/system/class-signals.md` | **load when the task is orient / suggest / "what next?"** — request analysis, property derivation, coverage, or recommending a class/phase/flow the human did not ask for. It is the catalogue: surface-property vocabulary → **45 vuln classes**, signal → **the recon skills**, situation → **10 flows**, one line each. You cannot recommend what you do not know exists — but for an **exactly named skill** you do not need the whole catalogue, so skip it and save the context. A map, never a substitute for the file it points at. |
| 4 | `targets/<named>/index.md` | the manifest of **the target the human named** — nothing else |
| 5 | `targets/<named>/target.md` · `scope.md` | profile · **what the hunter said they are testing.** `scope.md` always loads because it is context you reason with — a model that enumerates can widen one apex into three thousand names. It is ~30 lines. It is **not** a gate: see *Scope is the hunter's* below. |
| 6 | `accounts.md` **only when the task involves a principal** | authenticated recon, authz, IDOR, multi-tenant, privesc. Unauthenticated recon does not need it and should not pay for it. |
| — | `scope-evidence.md` **only on a dispute** | where the scope statement came from. You consult provenance when something disagrees, not every turn. |

**Which target?** Whichever the human names — "run subdomain enum on clover", "test IDOR on
acme's profile API". If exactly one exists, use it. If they name a *host* or paste a request,
match it against each `targets/*/scope.md` to work out **which brain it belongs in**: two matches
→ ask which; no match → say so and ask where to file it, because writing one engagement's recon
into another's brain is unrecoverable without version control. That is a question about
bookkeeping, not authorisation. **Otherwise ask.** Never carry over the last target you touched.

**Do not preload `skills/` or `playbooks/`.** Read the one skill the human invoked, plus the one
reference it names, when they invoke it. `class-signals.md` loads only when you are orienting or
recommending (above) — for an exactly named skill it is not needed, which keeps the resident tier
to `AGENTS.md` + `orchestrator.md` + the named target's small files.

---

## The router — say the thing, don't cite the path

| the human says something like | load |
|---|---|
| "resume acme" · "where were we" · "continue hunting acme" | `tools/brief.py <t> --resume` — unfinished work, standing decisions, stale inputs, the exact read set |
| "initialize a new target" · "start a new program" · any requested work naming a target with no directory | `skills/setup/init-target.md` — scaffold from the supplied scope. An explicit initialization stops after scaffolding; if initialization was only a prerequisite to a named task, continue with that task after the required scope decisions are settled. |
| "run subdomain enum" · "http probe the estate" · "prioritize the surface" · any named recon objective | `skills/shared/_recon-method.md` **+ the one categorized body resolved by its slug** — `tools/preflight.py <t> --list` groups and names them all; `skills/README.md` is the human map |
| "test for IDOR on /api/orders" · "check access control" · "look for logic bugs" | `skills/vulnerability-testing/vuln-hunt.md` → `playbooks/vulns/_master-vuln-hunting.md` **+ one class file** |
| "here's a request from Burp, analyse it" | `skills/analysis/analyse-request.md` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tarekmo0/HunterStick](https://github.com/tarekmo0/HunterStick) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
