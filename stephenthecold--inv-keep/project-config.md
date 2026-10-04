---
trigger: always_on
description: Per-session ramp-up. **Read this first**, then [docs/PROJECT_STATE.md](docs/PROJECT_STATE.md)
---

# Working on Inv-Keep with Claude

Per-session ramp-up. **Read this first**, then [docs/PROJECT_STATE.md](docs/PROJECT_STATE.md)
for the full architecture digest. CHANGELOG.md tells you what each version added; this
file tells you how to actually *work* in the repo without re-learning the foot-guns.

## Verification recipe (run this on every meaningful change)

```bash
# Python syntax (uses the working venv at C:\temp\inv-keep-venv on this machine —
# the project's own ./.venv is a broken Linux symlink farm from OneDrive sync).
/c/temp/inv-keep-venv/Scripts/python.exe -m py_compile app/*.py

# JS syntax
node --check app/static/app.js

# Live UI verification — the Claude Preview MCP launch config is at the WORKSPACE
# root (../.claude/launch.json), NOT inside inv-keep/.  Starts uvicorn on :8096
# with data/preview.db.  See preview_start, preview_screenshot, preview_eval.
```

CI runs the equivalent on every push (`.github/workflows/ci.yml`) plus a growing
regression suite: cart-flow end-to-end, NaN-geo rejected, CSRF rejected without
token, XSS payload kept as JSON-escaped data, kiosk PIN lockdown enforces the
view/view_catalog/checkout floor (admin paths 403, catalog browse 200), per-item
stock-modal payload renders on `/parts?cat=all`, per-location stocktake adjusts
counts + writes audit rows. **When you change a URL or perm, expect to update
CI assertions too** — that's how every CI failure since v1.17 has played out.

## Known gotchas on this Windows + OneDrive box

- **Watchfiles + OneDrive miss new-file create events**. After a `Write` of a new
  module (vs an `Edit` of an existing one), restart the preview server cleanly:
  `preview_stop` → delete `app/__pycache__` → `preview_start`. Otherwise you'll see
  `ImportError: cannot import name 'foo' from 'app'` even though the file exists.
- **PowerShell drops Secure cookies over plain HTTP**. The session cookie is
  `Secure=True` unless `DISABLE_AUTH=1`. So PowerShell-driven smoke tests against
  `http://localhost:8000` get CSRF-rejected — POST handlers never see the session.
  Either set `DISABLE_AUTH=1` in `.env` for local smoke OR test through the browser
  via the Preview MCP.
- **The project's `.venv/` is unusable** on this machine (OneDrive turned the Linux
  symlinks into 8-byte text files). Use `C:\temp\inv-keep-venv` instead, or rebuild
  the venv freshly inside the project on a non-OneDrive disk.
- **gh CLI and Docker aren't on the default PATH.** Use the full paths:
  - `C:\Program Files\GitHub CLI\gh.exe`
  - `C:\Program Files\Docker\Docker\resources\bin\docker.exe`

## Known gotchas in production / on the docker host

- **`data/` ownership.** The container runs as uid `10001` (see Dockerfile);
  the bind-mounted `./data` host directory MUST be writable by that uid or
  every write throws `sqlite3.OperationalError: attempt to write a readonly
  database` — and the user-visible symptom is a 500 on whatever POST tries
  to flush an audit-log row (settings saves were the first place we saw
  this). Fix on the host: `sudo chown -R 10001:0 ./data && sudo chmod -R
  u+rwX ./data && docker compose restart inv-keep`. `install.sh` does the
  chown for fresh installs; restores / manual file ops can re-flip it.
- **GitHub Releases creates the tag, not the other way round.** The
  release pipeline (`.github/workflows/release.yml`) triggers on push of
  any `v*` tag and publishes a multi-arch image to GHCR. The Releases UI
  is the ONLY way to create the tag from the browser: Draft new release →
  type the version into "Choose a tag" → click **"+ Create new tag:
  vX.Y.Z on publish"** → Publish. Editing the *title* of an existing
  release does nothing — the workflow doesn't fire and `:latest` on GHCR
  stays put.
- **The tag name is typed by hand into a text box, and nothing validates
  it.** Every release mishap in this repo so far has been a typo in that
  box, and each fails differently:

  | Typed | What happened |
  |---|---|
  | `v,1.40.0` | stray comma — still matched `v*`, so it published under a junk tag |
  | `v1.14.1` | transposed digits for what the CHANGELOG calls v1.41.1 — published under the wrong version |
  | `V1.42.0` | **capital V** — `tags: ['v*']` is case-sensitive, so the pipeline never fired at all |

  Lowercase `v`, then the exact version from `app/version.py`. Nothing
  downstream will correct you.
- **Verify the release actually published — two checks, both needed.**
  1. The tag exists *and* has the right case. Grep case-insensitively so a
     wrong-case tag shows up instead of looking absent:
     `git ls-remote --tags origin | grep -i 'X\.Y\.Z'`
  2. The **Release image** workflow ran with event `push` (not
     `workflow_dispatch`) on that tag, and its "Derive image tags" step
     lists `:vX.Y.Z` + `:vX.Y` + `:latest`.
- **A manual `workflow_dispatch` run is NOT a substitute for the tag
  push.** It's tempting when you notice the pipeline didn't fire, and it
  looks like it worked — the run goes green and `:latest` does get
  updated. But `docker/metadata-action` derives tags from `github.ref`,
  not from the `ref` input, so a dispatch on `main` produces only
  `:main`, `:latest` and `:sha-<short>`. The `type=semver` patterns never

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stephenthecold/inv-keep](https://github.com/stephenthecold/inv-keep) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-04 -->
