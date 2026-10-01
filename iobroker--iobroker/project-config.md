---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

This is **not** the ioBroker platform itself — it is the **installer/maintenance tooling** for it. The
ioBroker runtime lives in separate repos (`iobroker.js-controller`, `iobroker.admin`, …) and is pulled in
from npm at installation time.

One source tree feeds two delivery channels:

- **Linux/macOS/FreeBSD**: bash scripts, built into `dist/` and uploaded via SFTP to `https://iobroker.net/`.
  End users run `curl -sL https://iobroker.net/install.sh | bash -`.
- **Windows**: the `lib-npx/` Node.js package, published to npm as `@iobroker/install` (and, with a renamed
  `package.json`, as `@iobroker/fix`). End users run `npx @iobroker/install`.

## Commands

```bash
npm install                 # ~30 s
node tasks --create         # build dist/install.sh, dist/fix.sh, dist/diag.sh, dist/node-update.sh
node tasks --deploy         # SFTP upload of dist/ (needs SFTP_HOST/PORT/USER/PASS env vars)
npm run deploy              # = create + deploy (what the release workflow runs)
npm run make-fix            # rewrites package.json name to @iobroker/fix, for the second npm publish
node test.js                # polls http://localhost:8081 for "<title>Admin</title>", 10x with 5 s waits
```

Useful env vars for `tasks.js`: `DEBUG=true` (verbose SFTP), `FAST_TEST=true` (connect but simulate the
upload instead of writing).

### Testing

There is **no unit test suite**. `mocha`/`chai` are devDependencies but no spec files exist, and
the CI step that used to install them is gone. Verification is end-to-end only:

```bash
node tasks --create && bash ./installer.sh --silent   # 6-10 min; do not cancel, use 15+ min timeouts
bash .github/testFiles.sh                             # asserts ownership/permissions under $IOB_DIR
curl -s http://127.0.0.1:8081 | grep '<title>Admin</title>'
```

The service takes 1-2 minutes to come up after the installer finishes.

**`installer.sh` and `fix_installation.sh` download `installer_library.sh` from `master` at runtime**, so a
plain `bash ./installer.sh` does not exercise local library changes. To test them, build the self-contained
artifact first and run that:

```bash
node tasks --create && bash dist/install.sh --silent
```

CI does the same: `test.yml` builds `dist/` and runs `dist/install.sh`, so library changes on a branch are
what actually gets tested. (Until that was fixed, a `sed`-strip step that never worked meant CI ran
master's library, and a PR touching only `installer_library.sh` got a green run that executed none of its
changes.)

The same used to apply one level up, to `versions.json`, which the library downloads from `master` at
runtime: a PR editing it was exercised against master's values rather than its own, and contradicted the
matrix, which is built from the local file. `VERSIONS_URL` now falls back to the GitHub URL instead of
hardcoding it, and both `Install ioBroker` steps point at the checkout:

```yaml
env:
  VERSIONS_URL: file://${{ github.workspace }}/versions.json
```

So a change to `versions.json` is verified before it is merged. In `node-update.sh` the default is
applied *before* `readonly` — the other order silently discards a value from the environment.
The variable is for testing; end users have no reason to set it.

### Linting

`npx eslint` does **not** work. The repo has an ESLint 8-style `.eslintrc.json` and no `eslint.config.js`,
while `eslint` 9.x is the installed devDependency. Do not add lint steps expecting it to run.

## Architecture

### The bash build pipeline (`tasks.js`)

`installer.sh` and `fix_installation.sh` each contain a block delimited by
`# get and load the LIB => START` / `# get and load the LIB => END` that, at runtime, curls
`installer_library.sh` from GitHub and sources it. `node tasks --create` **replaces that block with the
literal contents of `installer_library.sh`**, producing self-contained `dist/install.sh` and `dist/fix.sh`.
`diag.sh` and `node-update.sh` have no library dependency and are copied verbatim.

So: shared bash helpers (platform detection, package install, user creation, Node.js install, permissions,
Redis) belong in `installer_library.sh`; each entry-point script keeps only its own flow.

### The deployment loop — why edits do not take effect locally

The installed `iob` wrapper does **not** call scripts from this repo. It downloads them from
`https://iobroker.net/` at invocation time (`FIXER_URL`, `DIAG_URL`, `NODE_UPDATER_URL` in
`installer_library.sh`). A change to `fix_installation.sh`, `diag.sh` or `node-update.sh` reaches users only
after a GitHub release triggers `deploy.yml` → `npm run deploy` → SFTP. Test locally by running the script
directly (`./fix_installation.sh`), not via `iob fix`.

### `iob` / `iobroker` wrapper

`installer.sh` generates an executable at `$IOB_DIR/iobroker`, symlinked into `/usr/bin` (or
`/usr/local/bin`) as both `iob` and `iobroker`. It:

- routes `start`/`stop`/`restart` (exactly one argument) to `systemctl`, `launchctl`, or init.d;
- intercepts `fix`, `diag`, `nodejs-update` and downloads + runs the remote script as `$IOB_USER` —

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ioBroker/ioBroker](https://github.com/ioBroker/ioBroker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
