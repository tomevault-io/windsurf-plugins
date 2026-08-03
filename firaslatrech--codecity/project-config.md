---
trigger: always_on
description: A repo rendered as a 3D city you drive through. Building = file (height = √lines, footprint ∝ bytes^0.72), district = folder, glowing windows = high-churn files (≥ p90 commits). Drive-only: arcade car with chase cam, bottom card inspects the nearest building (lines/size/commits/age/co-authors), E opens the file in-city. Three cars (coupe/racer/truck, C or the button cycles), each with its own handling and synthesized engine voice. T = git-history timelapse: auto-orbit camera + scrubber, the city
---

# CodeCity

A repo rendered as a 3D city you drive through. Building = file (height = √lines, footprint ∝ bytes^0.72), district = folder, glowing windows = high-churn files (≥ p90 commits). Drive-only: arcade car with chase cam, bottom card inspects the nearest building (lines/size/commits/age/co-authors), E opens the file in-city. Three cars (coupe/racer/truck, C or the button cycles), each with its own handling and synthesized engine voice. T = git-history timelapse: auto-orbit camera + scrubber, the city grows commit by commit (playback is activity-weighted, 15s full run). P = share card: cinematic 1600×900 PNG (city + stats + contributor faces + watermark) and an 8s WebM orbit clip where the city regrows (MediaRecorder, no deps). `?shot` renders a deterministic fixed-camera still for screenshot diffing; `?nobloom` skips post-processing.

## Pipeline

```
npm install                       # once
npm run dev                       # vite on http://localhost:8137
```

Routing: `/` is a landing page that takes a GitHub URL; submitting hits `GET /analyze?url=` (middleware in `vite.config.js`) which fetches the repo into `.cache/<owner>__<repo>` (cached forever — delete to re-pull; empty dir = interrupted fetch, auto-refetched), runs the analyzer, bakes `public/cities/<repo>.json`, and the page navigates to `/<repo>`. There is NO full clone: the tree comes as a streamed tarball (codeload) and history as a parallel `--bare --filter=blob:none` clone into `<dir>/.git` (+ `core.bare false`) — commits and trees only, no file contents, since `git log --name-only --no-renames` never reads blobs. `/<name>` loads `/cities/<name>.json` and drives it; unknown names bounce back to the landing page. Local repos skip the server: `node analyze.mjs <repoPath>` bakes the same file, then open `/<name>`.

AI city plan (optional): if `GROQ_API_KEY` is set when the dev server runs, `/analyze` also asks Groq (llama-3.3-70b) to pick 4–6 landmark files + a city motto from a stats digest; baked as `city.plan`, rendered as floating golden signs + a motto at the gate. No key, any failure, or a big city (> 1500 buildings) → `plan: null`, city renders normally. The key stays server-side.

Big-repo limits (all ponytail ceilings, raise if needed): cities keep only the ~3500 biggest files (smallest pruned before layout), git history parsing caps at the 50k most recent commits, line counting is a single byte-scan (no utf8 decode), avatar API pages fetch in parallel. Repos the GitHub API reports > ~1GB skip the history download entirely (even blob-less, linux's 1.3M commits of metadata are GBs) — those cities have no glow/timelapse; clone locally + `node analyze.mjs` for a monster WITH history. The vite watcher ignores `.cache/` and `public/cities/`.

Scale guards in `/analyze` (for a public deploy): `GITHUB_TOKEN` in `.env.local` lifts the API limit 60→5000/h (unauth is per-IP, ~15 analyzes/h); in-flight dedup (concurrent hits on the same repo share one bake via an `inflight` Map); `MAX_ACTIVE=4` concurrent bakes else a 503; the analyzer runs via `node analyze.mjs --bake <dir> <name> <avatarsJson>` in a child process (execFile) so its sync git-log + walk never blocks the event loop; `MAX_CACHED=40` LRU eviction of clone dirs by mtime (baked city.json kept). Avatars are written to `<dir>/.cc-avatars.json` for the child; the analyzer's walk skips dotfiles so it never becomes a building.

The renderer never touches git or the filesystem — everything it needs is baked into the city JSON by the analyzer, including the treemap layout. Keep that boundary. Server-side pieces live only in `vite.config.js`: `/analyze` (validated owner/repo, execFile — never a shell) and `/raw?repo=&p=` (file contents for the in-city reader, whitelisted against the baked city).

## Files

- `analyze.mjs` — stdlib-only Node module: fs walk, squarified treemap, one-pass `git log` enrichment (commits + age + top-3 authors with avatar URLs per file; author identities merged across emails; timelapse data: per-building `born` + 32-bucket commit histogram `g` + `city.timeline`). Exports `analyze(repoPath, name?, emailAvatars?)` for the server; CLI bakes `public/cities/<name>.json`. No shebang — esbuild bundles it into the vite config and chokes on mid-bundle shebangs. Self-test: `node analyze.mjs --check` — must pass after any analyzer change, then rebake.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FirasLatrech/codecity](https://github.com/FirasLatrech/codecity) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
