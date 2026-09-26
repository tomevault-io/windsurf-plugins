---
trigger: always_on
description: ⚠️ **STILL READ APP STORE CONNECT BEFORE YOU TRUST THIS.** `node
---

# Radiant — read this first, every turn

## THE iPHONE APP — 1.1 (build 21) READY FOR SALE, approved 2026-09-18

⚠️ **STILL READ APP STORE CONNECT BEFORE YOU TRUST THIS.** `node
scripts/asc.mjs get 6804891721` prints the live state in one line; this
heading has gone stale three times. 1.0 (build 7) was approved 2026-09-16; 1.1
(build 21) was submitted 2026-09-17 from the command line (`asc.mjs submit`)
and approved the next morning with no questions.

**1.1 carries:** Hugging Face search with a run/won't-run verdict (unfiltered
— TG-454, do not reinstate a word filter), Archive on chats, the keyboard fix,
the unsent-message fix (TG-467), the byline link to templetontech.com, and an
age rating of 17+ — answered honestly for an open model list. `CURRENT_PROJECT_VERSION`
is 21; the next upload must be higher.

**The next build is an update:** a new version number in App Store Connect
(`asc.mjs new-version`), its own what's new, its own review. Screenshots are
still 1.0's — replace them with the next submission (`asc.mjs shots` counts
them). `scripts/asc.mjs` can do everything short of signing in.

**Anything to do with the submission: use the `app-store-review` skill**
(`.claude/skills/app-store-review/`, also installed at `~/.claude/skills`;
published at https://github.com/templetongroup/app-store-review — the repo is
the copy people install, so a change here goes there too). It is the whole
adventure — both rejections, the TestFlight false alarm, the privacy Publish
button, the two-button resubmit — turned into a protocol.

**The catalogue is published, not only compiled in.** `apps/ios/catalog.json` is
fetched at launch and applied over the built-in Swift array, so a broken row can
be corrected in minutes instead of a review cycle. It is GENERATED from that
array (`npm run catalog:export`), so the two cannot drift, and every failure
falls back to what shipped.

⚠️ **That also means a bad publish reaches every phone at once.** `npm run
catalog:publish` runs the export, then `scripts/catalog-check.py`, which probes
every repo and refuses on undeclared quantization, a size more than 10% off the
real blob total, or a 404. Do not copy catalog.json to the website by hand.
**Before any future submission, run `npm run catalog:check`.** It fails any repo
under ~1.2 bytes per parameter that declares no quantization — the Gemma 4
defect, which shipped because the old check only asked whether MLX implemented
the architecture. It also builds every row's real config.json with the phone's
engine (the pinned one, and for main-list rows the oldest one still on phones —
App Store 1.1's), which is what would have stopped Nemotron 3 Nano 4B shipping.

Radiant is Tony's own coding harness: an Electron app wrapping a local node
server (`server/index.js`, port 5834) and a React UI (`src/`). It is a public,
MIT-licensed repo, signed and notarized, and it auto-updates from GitHub
Releases. Work on `master`.

## Written is not shipped

**Every change closes all three of these, in the same turn:**

1. **Git** — committed with a real message, and pushed. Tony runs the packaged
   app, not the dev server, and other agents work from other checkouts. An
   uncommitted fix looks exactly like no fix: on 2026-08-22 six corrected files
   sat in the working tree while he tested the release and reported the bug as
   still broken.
2. **The in-app Read me** — the `GUIDE` array in `src/components/Settings.jsx`
   (Settings → "Read me"). Standing rule from Tony: *"you MUST update that
   readme when features are added or changed. end users deserve that."* Write it
   for someone using the app: what they can now do, plain language, US spelling.
3. **Linear** — team **The Templeton Group** (TG), project **Radiant**. Ship
   something → its issue goes to Done, or create one already Done. Spot a
   problem you are not fixing → file it.

**This is automatic, not a question to ask.** Tony has standing authorization:
run the `ship-sync` agent at the end of any turn that changed behavior.

Run the objective half and fix whatever it flags:

```bash
node scripts/ship-check.mjs
```

It verifies committed / pushed / Read-me-kept-current / tagged — and, fifth,
**judged**: `scripts/ship-judge.mjs` has Jev (a decision model, see
`server/decide.js`) read the commit message and any new Read me entries and
answer whether they say *why* and whether they are written for a person using
the app. On its first run it failed two entries that talked about prompt
caches and tool schemas to users. Below 50% fails the check; unreachable, it
passes and says so. Rewrite in plain words, commit again. Since 2026-09-19 it
also reads the DIFF, per source file: a likely bug or a security hole at
≥ 70% fails; "users would notice, message is silent" and "logic with no test"
print as ⚠ warnings for a person to weigh.

New GitHub issues are sorted the moment they land (`scripts/triage.mjs`, run by
`.github/workflows/triage.yml`): app, kind, severity, possible duplicate — as
labels, with the probability, and nothing under 60% (90% for duplicate) is
applied. `node scripts/triage.mjs --closed --dry-run` shows what it would say. Or hand the
whole job to the **`ship-sync`** agent (runs on Haiku, cheap) — it loops until

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [templetongroup/radiant](https://github.com/templetongroup/radiant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
