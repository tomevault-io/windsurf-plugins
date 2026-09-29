---
trigger: always_on
description: This file is the authoritative reference for Claude Code when working on this repository.
---

# design-agent-skills — Contributor Guide

This file is the authoritative reference for Claude Code when working on this repository.
Follow it exactly — the test suite enforces most of these rules automatically.

---

## What this repo is

A two-tier catalogue of design skills for Claude Code, Cursor, Codex, OpenCode, and 30+ other AI agents.

```
Tier 1 — Routing layer (6 routers, permanent)
  design-catalogue → domain catalogues → implementation skills

Tier 2 — Implementation pointers (128 stubs)
  Lightweight stubs that tell an agent what a skill does and how to fetch the real one.
  On first use the agent upgrades the pointer to the full skill in-place.
```

Skills install via: `npx skills add podo/design-agent-skills [-g]`
This fetches directly from GitHub — **npm publish is only needed for changes to `bin/cli.mjs`**.

---

## Repository layout

```
skills/<name>/
  stub.yaml     — machine-readable metadata (type, tier, upstream)
  SKILL.md      — human/agent-readable pointer with frontmatter + install instructions

test/
  stubs.test.js — run with: npm test
  cli.test.js   — bin entry tests

bin/cli.mjs     — thin wrapper around `npx skills add`
VERSION         — semver, single line
package.json    — version field must match VERSION
CHANGELOG.md    — keep-a-changelog format
README.md       — skills table, router table, supply chain section
```

---

## Adding a skill — checklist

1. Evaluate the skill (see criteria below)
2. Create `skills/<name>/stub.yaml` — set `rank: 3` by default; promote if clearly rank-1 or rank-2
3. Create `skills/<name>/SKILL.md`
4. Update `skills/design-catalogue/SKILL.md` — add to the right section + routing guide
5. Update the relevant domain catalogue SKILL.md
6. Update `README.md` — skills table row + supply chain tier count if community changes
7. Run `npm test` — must pass 100%
8. Bump version (see versioning section)
9. Commit and push to `main`
10. Push the version tag (`git tag v<version> && git push origin v<version>`) — CI creates the release with the skills zip and publishes to npm automatically

---

## Fixing bugs — checklist

1. Reproduce the bug and identify the root cause
2. Fix it
3. **Add a regression test** that would have caught the bug before the fix:
   - Data bugs (missing fields, wrong values in stub.yaml / SKILL.md) → `test/stubs.test.js`
   - CLI behaviour bugs (wrong counts, wrong filtering, wrong commands) → `test/cli.test.js`
4. Run `npm test` — must pass 100%
5. Bump the patch version (see versioning section)
6. Commit the fix and the test together

Do not close a bug fix without a test. The test is proof the bug is fixed and stays fixed.

---

## Evaluation criteria — what makes a good skill

### Accept if all of these are true

| Signal | Why it matters |
|--------|---------------|
| Has a `SKILL.md` at the upstream repo root (or known path) | The pointer's upgrade target must exist |
| Installable via `npx skills add owner/repo` OR has a clear alternative install | Agents must be able to fetch the real skill |
| Covers a gap not already in the catalogue | No point in two near-identical skills |
| Addresses a real design/engineering use case | Scope must fit: UI, motion, a11y, Figma, data viz, content, etc. |

### Strong positive signals

- 50+ GitHub stars (traction, maintenance signal)
- Official org account (Anthropic, Vercel, Google, Expo, GSAP, LottieFiles…) → `tier: official`
- Active commits in the last 6 months
- Multiple sub-skills or slash commands (high density of capability)
- Unique angle not covered elsewhere in the catalogue

### Skip if any of these are true

- Archived repository
- `curl | bash` only install, no `npx skills add` pathway, and no reasonable `type: package` alternative
- Heavy overlap with an existing catalogue entry and no meaningful differentiation
- Not a SKILL.md — MCPs, web tools, VS Code extensions, pip packages go elsewhere
- Lives in a completely different ecosystem (pip/skilz, VS Code marketplace, etc.)
- 0 stars and fewer than 5 commits (too immature / likely abandoned)

### Tier assignment

| Tier | When to use |
|------|-------------|
| `official` | Published by the company/project that owns the upstream (Anthropic, Figma, Vercel, Expo, GSAP, LottieFiles, shadcn, remotion…) |
| `community` | Published by an individual or third party; 1+ stars; actively maintained |
| `experimental` | Very new, 0 stars, or unverified upstream — excluded from default installs |

### Rank assignment

`rank` controls which install profile includes this skill (`npx design-agent-skills --picks / --essentials / --all`).

| Rank | Profile | Criteria |
|------|---------|----------|
| `1` | **Picks** | Best-in-class for its category — one winner per domain. High stars, official when available, no redundancy. |
| `2` | **Essentials** | Adds real coverage without duplicating rank-1. Covers sub-niches, secondary tools, solid community picks. |
| `3` | **Extended** | Niche, experimental, heavy overlap with rank-1/2, platform-specific, or low star count. |

**Rules:**
- Routers (`type: router`) are **exempt** from rank — always installed.
- Every other stub **must** have a `rank` field (test enforces this).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [podo/design-agent-skills](https://github.com/podo/design-agent-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-29 -->
