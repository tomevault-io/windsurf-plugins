---
trigger: always_on
description: Every session in this repository begins by reading this file, then the boot lane in the order the generated block at the end of this file prescribes (the generated static lane `vibevm/vibespecs/boot/STATIC.xml` — or `STATIC.md` in a Markdown-lane project — first and in full, then every file `vibevm/vibespecs/boot/INDEX.md` names), then any relevant PROP/FEAT documents under `vibevm/vibespecs/common/` and `vibevm/vibespecs/modules/` for the task at hand. An owner-facing central session additional
---

# vibevm — read this first

Every session in this repository begins by reading this file, then the boot lane in the order the generated block at the end of this file prescribes (the generated static lane `vibevm/vibespecs/boot/STATIC.xml` — or `STATIC.md` in a Markdown-lane project — first and in full, then every file `vibevm/vibespecs/boot/INDEX.md` names), then any relevant PROP/FEAT documents under `vibevm/vibespecs/common/` and `vibevm/vibespecs/modules/` for the task at hand. An owner-facing central session additionally resolves its exact user-local context under `~/.vibe/steward/` as the installed multi-user-planning flow prescribes; a worker packet never claims that context. Only after that, start work.

**A task carrying `##subagent-quiet-clause` does not run that sequence.** A delegated worker, a review-only agent or a consulted subagent reads exactly the files its packet names — including whichever standing rules bind that particular task, named file by file — and its packet is the rest of its instruction surface. Selecting those files is the packet author's job, not the worker's: a worker that needs a rule its packet did not name reports a packet defect rather than reading the lane itself. The full lane costs roughly 145k tokens before such a session reaches its first instruction, and nearly all of it governs decisions a worker never makes.

The repository's commit-and-push discipline — human-authored **attribution** (never mark any part of this repository as AI-authored), **Conventional Commits**, **atomicity**, and commit **autonomy** (routine proceeds; non-routine stops and asks) — is the `git-practices` family, a dependency of this project loaded first and verbatim from the generated static lane in `vibevm/vibespecs/boot/`. The rules live in that static lane, not restated here. Authoritative record: [spec://org.vibevm.core/vibevm/common/PROP-000#commits](vibevm/vibespecs/common/PROP-000.xml#commits).

Authoritative record: [spec://org.vibevm.core/vibevm/common/PROP-000#commits](vibevm/vibespecs/common/PROP-000.xml#commits).

## Delegation-first — spend Claude on judgment, run execution on fractality

**The directive is now installed.** The standing posture — Claude's context and reasoning are the scarcest, most expensive resource in the room; the cheap worker slots sit idle, already paid for, so **delegate execution by default and keep Claude for architecture, planning, judgment, and review** (a session that codes, bulk-edits, or reads-and-summarizes work a worker could do is spending the very budget this directive exists to save) — is the `delegation-first` flow, a static dependency of this project. It carries the directive in full — the scarce-resource thesis and the ~5%-boss / ~95%-worker target, delegate-by-default, GLM-5.2 as the `big` worker slot, first-level swarm and RLM handling, the never-delegate set, and the obligations (always review; surface the analysis out loud; announce the harness). The decidable calculus it sits above — *delegate when verification is cheaper than generation*, scored on four axes (error cost / context / verifiability / size) with the verdict steps and per-model playbooks — is the `delegation-rules` flow it pulls, now **installed** as a dependency rather than read in-place: `spec://org.vibevm.fractality/delegation-rules/flows/delegation-rules/DECISION-MATRIX#root`.

What follows is **only** vibevm's operational specifics on that directive — the exact fractality entry points, how Rules 1 & 4 bind delegated work, and the live operating-facts ledger. The directive itself — delegate by default, GLM-5.2, RLM, swarms, review, surface, announce — is the package above, not repeated here.

**Running fractality here.** The first-level usage lives in the package; the
verified operating facts (profiles, tokens, packet schema, build state) are the
ledger below. The entry points between them: the launcher is
`vibevm/vibepacks/org.vibevm.fractality/fractality.ps1` (PowerShell) / `fractality.sh`
(Bash), built once via `cargo build -p fractality-cli` from
`vibevm/vibepacks/org.vibevm.fractality/fractality/v1.0.0/` against the global
`~/.fractality` home. Drive it — `./fractality.ps1 run --packet <task.toml>`
(sync) or `spawn … ; wait <id>` (async); free `route` / `gate` helpers (no
daemon, no spend); no-packet interim route
`opencode run -m zai-coding-plan/glm-5.2 "<task>"`. RLM's need-gate is
`fractality gate …`; its recursive-descent machinery is
`vibevm/vibepacks/org.vibevm.fractality/fractality/v1.0.0/vibevm/vibespecs/plans/FRACTALITY-RLM-PLAN-v0.1.xml`
(Campaign 3 Stage B, maturing). On Claude Code, `ultracode` / the Workflow tool
cannot spawn GLM workers directly, so a swarm under them still routes through
fractality.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vibevm/vibevm](https://github.com/vibevm/vibevm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-07 -->
