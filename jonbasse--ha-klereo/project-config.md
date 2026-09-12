---
trigger: always_on
description: Home Assistant custom integration for the [Klereo Connect](https://connect.klereo.fr) pool
---

# CLAUDE.md — ha-klereo

Home Assistant custom integration for the [Klereo Connect](https://connect.klereo.fr) pool
management system. Cloud-polling, no local API. Ported from MrWaloo's
[Jeedom plugin](https://github.com/MrWaloo/jeedom-klereo). User-facing docs: [`README.md`](README.md).

## Memory (Hindsight) — ALWAYS FIRST

**Before any task**, `mcp__hindsight__recall` with a task-specific query (full rule + MCP endpoint
in the global CLAUDE.md). Bank is `fizbot`, shared across runtimes. Never call `retain` manually.

---

## Hosting — read this before touching any issue, PR or branch

> ⚠️ **The default branch is `master`, not `main`.** This is the only repo in the constellation
> where that holds. Every fleet recipe hard-coded on `main` — a `--base`, a `git diff main…`, a
> comparison point — is wrong here, and most of them fail on the wrong ref rather than loudly.

> ⚠️ **GitHub is NOT a locked tombstone here — it is the distribution channel, and it is live.**
> The fleet-wide rule ("Forgejo canonical, GitHub demoted to a push-mirror whose issues are locked
> tombstones") holds for hosting but **not** for this repo's GitHub issues. Measured 2026-08-10:
> `JonBasse/ha-klereo` on GitHub is public, unarchived, `has_issues: true`, last pushed
> 2026-08-08, and carries **53 issues — one of them open (#58) and filed by an external user**.
> That is not drift: `custom_components/klereo/manifest.json` advertises
> `issue_tracker: https://github.com/JonBasse/ha-klereo/issues` to every HACS install, so that is
> where users legitimately report bugs. **Reading GitHub issues here is correct**; blanket-applying
> "never `gh`" silently discards the only inbound channel this integration has.

| | Forgejo (canonical) | GitHub |
|---|---|---|
| URL | `forgejo.dragonlance.xyz/JonBasse/ha-klereo` | `github.com/JonBasse/ha-klereo` |
| Registered as | `backend: forgejo` in fizbot `src/fizbot_data/repos.yaml` | — |
| Issues | the owner's backlog — `fb-issue backlog ha-klereo`, `fb-issue new` | **inbound user bug reports**, read them |
| Releases | tags | **releases HACS installs from** |
| CI | `.forgejo/workflows/` — `lint`, `test`, `gitleaks` | `.github/workflows/validate.yml` — `hacs`, `hassfest` (#89) |

> ⚠️ **Issue numbers DIVERGE between the two — a bare `#38` is ambiguous.** Forgejo is at #83 and
> GitHub at #58 (2026-08-10); they were the same repo before the migration, so low numbers collide
> and resolve to *different* issues. Always qualify: `JonBasse/ha-klereo#38 (Forgejo)` or a full
> URL. `CHANGELOG.md` links Forgejo throughout.

Backend routing (which CLI, which token, never guess): load the `managing-forgejo` skill. Short
version — `tea` for everything issue/PR, **read and write** · `fj` when the body is Markdown-heavy
(`--body-file`) · `gh` **only** to read the GitHub-side user reports and releases.

---

## HACS status — custom repository today, default-catalogue submission OPEN

Users install by adding `https://github.com/JonBasse/ha-klereo` as a **custom repository**
(procedure in `README.md`). That is how every current user got it, and it keeps working whatever
happens to the submission below.

**Default-catalogue submission: [hacs/default#10263](https://github.com/hacs/default/pull/10263)**,
opened 2026-08-23, **12/12 checks green**, tracked in #102. Expect **months**, not weeks.

> Measured 2026-09-08: **802 open PRs are older than ours** — that is the real queue, since it
> sorts oldest-first — out of **1147** open in total, against **203 merged in the last 30 days**.
> At that rate the wait is on the order of **four months**, and the arithmetic assumes merges are
> drawn from the front of the queue, which is not verified. ⚠️ Re-measure rather than trust this
> number; the earlier "~720 deep" in this file was of an unstated kind and cannot be compared to it.
>
> ⚠️ **`gh pr view` shows `Action checks completed = FAILURE` on that PR and it is NOT a problem.**
> Two workflow runs fired on 2026-08-23: the first was **cancelled** (every one of its children
> reads `CANCELLED`) and its aggregate check failed as a consequence; the second, 56 seconds later,
> is green on all twelve. Sort the checks by `completedAt` before concluding — the rollup lists
> both runs side by side and the failure is the superseded one.

⚠️ It replaces `hacs/default#6025` (2026-03-07, closed as stale 2026-08-01 with an invitation to
reopen). **#6025 is NOT re-openable** — its branch had to be rebased and GitHub refuses to reopen a
PR whose branch was force-pushed, permanently. Anything saying otherwise is stale.

🔴 **Never delete `.github/workflows/validate.yml` while a submission is in the queue.** HACS
requires the actions to be passing **at review time**, not at submission time. Deleting that file on
2026-07-15 silently voided the requirement for the last six weeks of #6025's wait, with no signal
anywhere. See #89 and #102.

> **The `hacs` and `hassfest` jobs run on GitHub, and only there.** `hacs/action` validates
> github.com's API view of the repository, so it cannot validate a repo it sees only through
> Forgejo. The split is disjoint on purpose — `lint` + `test` + `gitleaks` on Forgejo,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JonBasse/ha-klereo](https://github.com/JonBasse/ha-klereo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-12 -->
