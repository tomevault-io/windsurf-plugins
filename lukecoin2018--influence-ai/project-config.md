---
trigger: always_on
description: Read this before doing anything. Every rule here exists because skipping it
---

# CLAUDE.md — influence-ai / InfluenceIT

Read this before doing anything. Every rule here exists because skipping it
caused a real problem.

---

## Who you're working with

Lukas is the founder, not a strong coder. Explain decisions in plain terms and
make the call on technical questions — but surface genuine strategic choices for
him to decide.

He verifies in production with screenshots. Trust those over any tool's report,
including your own. If your plan doesn't match his understanding, he'll say so —
that check has caught real errors, repeatedly.

---

## Non-negotiable workflow

### Step 0 — prove your ref, every session

```
git rev-parse HEAD
git rev-parse origin/main
git fetch origin && git rev-parse origin/main
git log origin/main..HEAD --oneline
git status --short
```

**The fetch is mandatory and the pre-fetch SHA proves nothing.** `origin/main` is
a cached local pointer; it means something only after fetching. A checkout has
been stale several times, and once a PR believed to be merged was not — so a fix
was thought live in production when it wasn't.

If HEAD doesn't match what the prompt expects, STOP. If `main` has moved *ahead*,
verify with `git diff --quiet HEAD origin/main` before assuming it matters — main
catching up is the benign direction.

**A stale checkout is not always harmless.** If files in scope changed in the gap,
reading them from disk gives wrong answers. Read blobs from the ref
(`git show origin/main:<path>`) for investigations, and make Lukas pull before
building.

**Watch for stray worktrees.** `git worktree list` — a detached worktree under
`.claude/worktrees/` once swallowed a `git pull` and made it look like the pull
hadn't run.

### Two phases, always

**Investigate → STOP for review → build → show the full diff → STOP.**

Never build before the diagnosis is approved. Never commit before the diff is
seen. Never merge or deploy — that's Lukas's, always. Opening the PR is his too;
give him the link.

### One concern per branch

No unrelated fixes riding along, however small. Note them in the report instead.

### Read-only means read-only

No writes, no edits, no build, no SQL, no browser. If you find something broken
during an investigation, write it down and keep going.

### Evidence, not assertion

Cite file paths and line numbers for every claim. **UNKNOWN is a good answer; a
plausible-sounding guess is not.** If the code can't tell you, say what would
settle it and don't run it.

Where you can cheaply turn a reading into a measurement, do. Running a helper and
reading its real output beats transcribing what you think it emits. That has
caught things a code review would have missed.

### Never enter credentials

Don't type passwords into forms, don't ask for a Postgres connection string. A
test needing a login goes back to Lukas.

---

## The VPS — three separate incidents came from undocumented infrastructure here

Vercel has been clean throughout. Every environment-specific bug has been the VPS.

### Deploying

```
cd /home/lukelmg/public_html/influenceit.app \
  && git pull origin main \
  && npm install \
  && rm -rf .next \
  && npm run build
```

Then restart **InfluenceIT** in the Webuzo dashboard.

- Four Webuzo apps run on that box. **InfluenceIT is port 30001.** `lmg.media` is
  30000, the scraper 30002, `creators.lmg.media` 30003.
- The old chain ended `fuser -k 30000/tcp && pm2 restart all`, which restarts a
  *different site*. InfluenceIT kept serving its old build from memory while
  `.next` was replaced underneath — producing `ChunkLoadError` and, for a whole
  day, a stale client running against a new server.
- pm2 manages only the old `lmgmedia` app. Its logs are not InfluenceIT's.
- Nothing supervises the Node process. Killing it does **not** respawn it; use the
  Webuzo dashboard.
- `rm -rf .next` before building is required.
- Hard-refresh after deploying. An incognito window is the cleanest way to tell a
  cache problem from a real one.
- Vercel green before VPS.

### nginx caches everything, and this caused a cross-user data leak

Config: `/usr/local/apps/nginx/etc/conf.d/webuzoVH.conf`

```
proxy_cache_key "$scheme://$host$request_uri"   # URI ONLY — no cookie
proxy_cache_valid 200 301 302 60m
proxy_cache_min_uses 1
```

Any 200 arriving with no `Cache-Control` is stored for an hour and replayed to
**every session**. That is how one creator's brand matches were served to
another. There is no `proxy_ignore_headers`, so nginx does honour `Cache-Control`
from the app — which is why the fix is application-side.

**The rule: every API route that reads a session must send**

```
Cache-Control: private, no-store, no-cache, max-age=0, must-revalidate
Vary: Cookie
```

Use `withNoStore()` from `lib/http/no-store.ts` — wrap the handler, don't add
headers per return. There were 63 return sites across 13 routes; wrapping covers
every branch by construction, including 401/403/504. A cached 401 would lock out
a valid session.

`export const dynamic = 'force-dynamic'` does **not** write a Cache-Control
header on the route-handler path — Next derives `isIsr` from the prerender
manifest, and the header is only written inside the ISR branch. Adding it looks
like a fix and does nothing.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lukecoin2018/influence-ai](https://github.com/lukecoin2018/influence-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
