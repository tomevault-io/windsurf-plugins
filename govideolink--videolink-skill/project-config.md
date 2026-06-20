---
trigger: always_on
description: >
---


# Videolink Agent Skill

Videolink is a video platform built for AI agents. You can record demos
of your own work, consume videos others have shared, search a library of
your org's videos, and share polished artifacts with humans and other
agents.

This skill tells you exactly how to do each of those things. It is
intentionally terse and recipe-shaped. Run the one recipe that matches
your current task.

## Before running any recipe: make sure you have the skill

Three ways to have this skill available, in order of preference.

### 1. Installed via `npx skills add` (preferred)

If the user's repo already has this skill installed at a standard
agent-skill path (e.g. `.claude/skills/videolink/SKILL.md`,
`.agents/skills/videolink/SKILL.md`, `skills/videolink/SKILL.md`,
or wherever your runtime looks), you already have a committed
snapshot and every future session auto-loads it — no fetch needed
per recipe.

**Check for updates once per session.** Committed skills are static:
they don't auto-refresh when we publish a new version. Once per
session (not per recipe — cache the result for the session):

1. Read the `version` field from the YAML frontmatter of the
   installed SKILL.md (e.g. `version: "3.0"`).
2. GET `https://api.govideolink.com/.well-known/videolink-skill.json` and note
   the `skill.version` field.
3. If they match: installed is up to date, nothing to do.
4. If the server's version is higher by semver: installed is
   behind. Offer an update (see "Offer skill update" in the "Ask
   the user" section). Don't block — proceed with the installed
   (stale) copy for this session if the user declines.

**Compare `version`, not sha.** Our publish pipeline rewrites a
few absolute URLs to relative paths when snapshotting the skill
into govideolink/videolink-skill (so e.g. `references/API.md`
resolves relative inside the installed repo). That makes the
installed SKILL.md's sha256 different from the sha the live
server publishes at `/.well-known/videolink-skill.json`. Version
is the authoritative "releases" signal and is the same across
both shapes; sha is specific to the serving shape and should not
be used for this check.

**Bonus: respect `skills-lock.json` if present.** If the user's
repo has a `skills-lock.json` at its root (skills.sh maintains
this), look up the `videolink` entry and check the `source`
field. If it's anything other than `govideolink/videolink-skill`
(e.g., the user is on a fork for a reason), **skip the update
prompt** — an update would pull govideolink's version over their
intentional fork. If `videolink` isn't listed in
`skills-lock.json` but SKILL.md is on disk, someone installed it
manually; the update check still applies.

**If the skill is NOT installed** AND the user is likely to use
Videolink more than once AND they have a repo you can commit to,
**offer to install it** (see "Offer persistent install" in the
"Ask the user" section below). The install command is one line:

```
npx skills add govideolink/videolink-skill
```

This clones govideolink/videolink-skill, copies SKILL.md +
references/ into the runtime's conventional skill directory, and
the user commits it. From that point on the skill is part of the
project. To update later: `npx skills update` (or
`npx skills update videolink` to target just this one).

### 2. Cached via `.videolink/skill.ref` (fallback)

When the user declines the install, or you're operating in a
checkout you can't commit to (e.g. a sandboxed CI run), run this
check at the top of every recipe (not at session start — it is
scoped to the recipe, not ambient).

1. Look for `.videolink/skill.ref` in the repo you are working on.
2. If it is absent, fetch this SKILL.md from the canonical URL, save it
   to `.videolink/SKILL.md`, and write `.videolink/skill.ref` with
   `{sha256, canonical_url, fetched_at, local_path}`.
3. If the ref is present AND `fetched_at` is within the last **24 hours**
   AND the ref's `sha256` matches the server's
   `https://api.govideolink.com/.well-known/videolink-skill.json` `sha256`: use the cached
   `.videolink/SKILL.md` and proceed.
4. If the ref is stale (older than 24 hours) OR the server's sha256
   differs: refetch SKILL.md, rewrite the ref, and continue.
5. On any network failure during steps 2/3/4: use the cached
   `.videolink/SKILL.md` if it exists and continue with a warning. If
   there is no cached copy, stop and tell the user.

Add `.videolink/SKILL.md` and `.videolink/skill.ref` to `.gitignore` so
you do not commit the cache. The ref file keeps your agent honest: if
the server publishes an updated recipe, you notice within 24 hours.

### 3. Live fetch (last resort)

If neither of the above works (no skill installed, no write access
to cache), fetch the canonical URL directly each time you need the
skill. Slow and bandwidth-wasteful compared to the other two paths —
only use as a last resort. Still works.

## Two ways to connect

The choice between Option A and Option B depends on **whether a human is
running you**, not just on where you're running. If a human is in the
loop, ALWAYS prefer Option A — you inherit their Videolink identity and
skip the whole agent-registration + org-claiming dance. Only fall through
to Option B when there is no one available to sign in.

### Option A — MCP inside a human's editor (preferred when a human is running you)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [govideolink/videolink-skill](https://github.com/govideolink/videolink-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
