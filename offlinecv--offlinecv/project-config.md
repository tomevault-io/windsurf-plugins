---
trigger: always_on
description: Guidance for Claude Code (claude.ai/code) working in this repository.
---

# CLAUDE.md

Guidance for Claude Code (claude.ai/code) working in this repository.

**This file is about writing code.** The *why* behind the process — merge-queue mechanics, the
AI-attribution setting, the full PII policy, deploy, license — lives in
`docs/CONTRIBUTING-PROCESS.md`. You do not need it to write a change. The three rules below are
the exception: they stay here because breaking them is silent, and for two of them, permanent.
Everything else that used to sit here is enforced by config or by a gate, and prose that restates
a setting only gives you something to reconcile.

## Hard rules (no exceptions)

- **Fixture PII.** Every PDF under `tests/fixtures/pdfs/` uses a synthetic persona: fake name,
  `@example.com` email, and a phone with a **real area code + `555` exchange + `0100`–`0199`
  subscriber** (e.g. `(312) 555-0123`). Not an area-code-`555` number like `(555) 010-0123` —
  `555` is an invalid NANP area code, so `libphonenumber-js` rejects it and the fixture's phone
  silently drops out of the score. An OSS template's shipped demo PDF is **not** an exception:
  Awesome-CV embeds posquit0's real CV, Deedy-Resume embeds Debarghya Das's. The repo is public;
  a leak means `git filter-repo` + a GitHub Support ticket.
  **`npm run check:fixtures` enforces part of this** (`scripts/check-fixture-pii.mjs`, wired into
  `verify` and CI). It checks every **PDF** under `tests/fixtures/pdfs/` — its text, its link
  annotations (`tel:`/`mailto:` hrefs) and its metadata — for four things: the email domain, the
  phone shape, a denylist of real people from OSS templates, and a metadata author. Since #654 it
  also sweeps the ground-truth sidecars (`*.truth.json`) — the one place in the repo that
  deliberately commits résumé field *values* as text. It exits
  non-zero naming the offending value. It does **not** check the other fixture types (png/jpeg/
  docx), and it **cannot** tell whether a *name* is synthetic — no check can. **That judgement is
  still yours.** Run it before adding a fixture or approving a PR that adds one, and also read
  `pdftotext <file>.pdf - | head -40` — the two cover different surfaces. `pdftotext` prints only
  the drawn page, so it cannot see a `tel:`/`mailto:` **link annotation** or the Info dict, and
  both have leaked here. Never trust the PR prose over the binary.
- **No AI attribution in git.** Claude Code has this off by config (`attribution` in
  `.claude/settings.json`); on any other harness it is on you. Never commit attribution
  trailers (like `Co-Authored-By:` or `Claude-Session:`) or PR badges.
- **One commit per PR.** `main` merges through a merge queue that derives the squash message from
  the branch, so a multi-commit PR lands `wip` and `fix lint` in `main` forever. Collapse the
  branch to a single commit before it reaches the queue.

## Project overview

offlinecv started as a browser-side PDF parser stress test for resumes and is growing into a **private, no-login job-search workbench**: drop a PDF in, see what a generic text extractor reads back, get an anonymous heuristic score, fix the resume in place (inline edit + on-device LLM rewrite), download a clean ATS-safe PDF, match it against a job description, and discover relevant job postings. The non-negotiable product constraint: **the PDF bytes and the resume text never leave the browser.** Scope the claim to *data custody*, not runtime — "runs on your device" is falsifiable and three egress paths ship today:

- **Job search** hits third-party feeds. What egresses is a short **keyword string** built from the user-editable query title + skills, never the resume text — `src/lib/job-search/providers/keywords.ts` is the sole resume-derived egress helper and the invariant the copy depends on. Company adapters egress only the **public company slug**.
- **JD URL fetch** (`src/lib/jd-match/fetch-jd.ts`, `fetchJdFromUrl`) reaches the third-party ATS page (Greenhouse / Lever / Workable / Recruitee / Ashby) when the user pastes a posting URL into the paste-a-JD disclosure on `/jobs/`. Requested by the user's own click; sends no résumé text, no keywords, no analytics — just the fetch of the public JD page itself.
- **Analytics** is env-gated PostHog (`src/lib/analytics.ts`, `VITE_POSTHOG_KEY`) — dead-code-eliminated when unset, but a hosted build ships it, so it is not the user's choice.

There is **no BYOK LLM provider in the tree** — `#320` is future; App.tsx / CapabilityStrip docblocks that mention BYOK are describing an unbuilt path. Don't cite BYOK as a current cloud path, and don't write a privacy line without grepping the actual `fetch(`/analytics/provider egress first.

### Product lanes and entry points

The build ships exactly two HTML entries (`vite.config.ts` `rollupOptions.input`):


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [offlinecv/OfflineCV](https://github.com/offlinecv/OfflineCV) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
