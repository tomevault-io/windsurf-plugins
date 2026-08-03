---
trigger: always_on
description: Guidance for Claude when working in this repository (File Browser, `filebrowser/filebrowser`).
---

# CLAUDE.md

Guidance for Claude when working in this repository (File Browser, `filebrowser/filebrowser`).

## Repo orientation

- Go backend (`github.com/filebrowser/filebrowser/v2`, ecosystem `go`) + Vue frontend under `frontend/`.
- The project is in **maintenance-only mode** (see `SECURITY.md`). Prefer small, surgical, well-tested changes.
- Version scheme: `v2.63.x`. Conventional-commit messages (`fix(scope): …`, `feat: …`, `chore: …`).
- Verify with `go build ./...`, `go vet ./...`, `go test ./...`. Reuse existing test harnesses (e.g. `signToken`, `scopedUserStorage`, `handle`, `customFSUser`, `mockUserStore`).

---

# Handling security advisories

Use this playbook when asked to triage, verify, fix, or manage GitHub security advisories.
All advisory state lives on GitHub and is driven with the `gh` CLI. States are
`triage → draft → published`, plus `closed`.

## 1. Fetch

```bash
# List by state (also: published, draft, closed)
gh api '/repos/filebrowser/filebrowser/security-advisories?state=triage&per_page=100' \
  --jq '.[] | {ghsa_id, severity, summary, state}'

# Full report for one advisory (read .summary and .description)
gh api /repos/filebrowser/filebrowser/security-advisories/GHSA-xxxx-xxxx-xxxx \
  --jq '.summary, "---", .description'
```

Always pull the **published** set too — you need it to dedup against.

## 2. Verify each report — do NOT trust the report text

Read the actual source **at HEAD** and reach one verdict per advisory:

- **CONFIRMED** — defect exists at HEAD. Quote the exact `file:line`.
- **FIXED** — already patched (find the fix commit).
- **FALSE / NOT APPLICABLE** — claim is wrong, or targets a different project.
- **NOT EXPLOITABLE** — pattern exists but no code path can reach the precondition.
- **DUPLICATE** — of a published advisory, or of another triage advisory.

Common traps — check each before accepting a report:

- **"Incomplete fix of a prior advisory."** Read the original fix commit and confirm the *specific*
  sibling code path is actually still unguarded — a prior fix may already cover a related path.
- **Wrong project.** Confirm the referenced files, symbols, and endpoints exist in this repo; reports
  sometimes describe a fork. If the cited code isn't here, close as not applicable.
- **"Legacy / upgraded / imported records are affected."** Trace the field's git history and confirm that
  some released version could actually produce such a record before believing it
  (`git log -S'Field' -- path`, `git show <commit>`, `git tag --contains <commit>`).
- **Overlapping reports.** Two triage advisories may share one root cause — consolidate and fix once.
- **Known, intentionally-unaddressed classes.** Some areas are known and tracked but not fixed (see
  `SECURITY.md`'s Known Issues); matching reports are duplicates.

Record, per advisory: verdict, the `file:line` evidence, exploitability preconditions
(default config? platform-specific? auth required?), and the disposition.

## 3. Fix (one commit per advisory)

- Branch off `master` first; never commit fixes directly to `master`.
- **One commit per fix**, each referencing the GHSA in the body (`Refs GHSA-xxxx-xxxx-xxxx`).
- When several advisories share a root cause because parallel code paths diverged, **centralize** the logic
  (e.g. `settings.CreateUserHome` used by signup, proxy, and hook provisioning) so they cannot drift again.
- Keep it surgical; match surrounding style.

## 4. Add a regression test per fix

- Reuse the existing harnesses; add a focused test asserting the fixed behavior (and that the legitimate
  path still works). Commit tests alongside the fixes (`test(scope): …`, `Refs GHSA-…`).
- Run `go test ./... && go vet ./...` before finishing.

## 5. Severity — set a CVSS vector, don't hand-assert

Set a CVSS v3.1 vector; GitHub derives the score **and** severity from it (overriding the plain `severity`
field). Encode the real preconditions in the vector so the band is defensible:

- Requires a specific target configuration/platform (e.g. case-insensitive FS) → **AC:H**.
- Unauthenticated vs needs an account → **PR:N / PR:L**.
- Keep it consistent with the **predecessor CVE's** rating (an incomplete-fix follow-up should not outrank
  its parent). Bands: `0.1–3.9` low, `4.0–6.9` medium, `7.0–8.9` high, `9.0–10.0` critical.

```bash
gh api -X PATCH /repos/filebrowser/filebrowser/security-advisories/GHSA-xxxx-xxxx-xxxx \
  -f cvss_vector_string='CVSS:3.1/AV:N/AC:H/PR:N/UI:N/S:U/C:H/I:H/A:H' \
  --jq '{ghsa_id, severity, score: .cvss.score, vector: .cvss.vector_string}'
```

## 6. Clean up the title

Rewrite reporter titles to the concise, sentence-case, backtick-free style of the published advisories
(state the vuln; drop "Incomplete fix of…" prefixes and jargon). The title is the `summary` field:

```bash
gh api -X PATCH .../security-advisories/GHSA-xxxx-xxxx-xxxx \
  -f summary='Proxy-auth auto-provisioning ignores createUserDir and grants the server root scope'
```

## 7. Rewrite the body into the standard structure

Reporter reports arrive in whatever shape the reporter used. Before drafting, rewrite the
`description` into the sections below — **reusing the reporter's own wording wherever it is

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [filebrowser/filebrowser](https://github.com/filebrowser/filebrowser) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
