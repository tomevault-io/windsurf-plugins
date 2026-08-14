---
trigger: always_on
description: - Merge upstream by published release tags as the stable baseline. Treat upstream `main` commits after the latest published release as uncertain until they are included in an upstream release tag or explicitly reviewed and accepted.
---

# Project Rules

## Upstream Merge Policy

- Merge upstream by published release tags as the stable baseline. Treat upstream `main` commits after the latest published release as uncertain until they are included in an upstream release tag or explicitly reviewed and accepted.
- Before merging post-release upstream commits, list the commits, classify the risk, and decide whether each commit should be included, deferred, or cherry-picked.
- When a post-release upstream commit is included, record why it is safe or necessary for this fork and verify it against the local/remote management boundary rules below.
- Upstream merges must preserve fork-owned product identity and release configuration. Do not let upstream changes overwrite `CC Switch Remote` branding, Tauri identifier, deep-link schemes, app data directory behavior, updater endpoint, updater public key, GitHub repository URLs, release asset names, helper release tags, signing validation, fork-specific packaging configuration, or project-facing README identity unless the user explicitly requests that change.
- Treat version and release metadata as controlled fork state, not as ordinary upstream drift. After every upstream merge, verify `package.json`, `src-tauri/Cargo.toml`, `src-tauri/Cargo.lock`, `src-tauri/tauri.conf.json`, release notes, and GitHub workflow version/tag behavior before committing or publishing. Upstream version bumps may be used as an input, but the fork release version must be chosen deliberately.
- Before pushing any upstream merge or release-preparation commit, run a mechanical fork-config diff against the previous known-good fork release. At minimum check product name, bundle identifier, updater endpoint, updater pubkey key id, release repository, helper artifact workflow triggers, and app data directory isolation. Record any intentional change; revert accidental upstream overwrites before continuing.
- Upstream merge validation is incomplete until both functional drift and configuration drift are checked. Passing tests, successful compilation, or conflict-free merges do not prove that fork identity, updater, packaging, or release behavior stayed correct.
- Never publish a release immediately after an upstream merge based only on code tests. First verify the release chain end to end: release tag points to the intended fork commit, artifacts are named for this fork, `latest.json` targets this fork repository, updater signatures match the configured fork public key, and local-only upstream release settings have not replaced remote/fork release settings.
- After an upstream merge, search runtime code and release scripts for upstream-owned repository URLs and wrong fork aliases before publishing. At minimum scan `src`, `src-tauri`, `scripts`, and `.github` for upstream release URLs, issue-only exceptions, stale helper repos, updater endpoints, release page constants, and install script defaults. Documentation links can remain upstream only when intentionally documenting upstream.
- Treat `README.md`, localized README files, repository description copy, sponsor/referral blocks, badges, screenshots, and project homepage links as fork boundary files. Upstream documentation changes in those files must be reviewed and either adapted to `CC Switch Remote` or reverted; do not publish with upstream-facing README identity by accident.
- Do not assume a newly built version validates the existing installed-version upgrade path. Before publishing a schema-changing or updater-changing release, test or reason through the old installed app version that users currently have: startup recovery page, update check command, update install command, restart behavior, and whether those commands work before the full application state/database is initialized.
- Recovery-mode UI commands must not require application state that is unavailable in recovery mode. Commands shown before database initialization, such as database-upgrade recovery, update checks, open release page, open config folder, and quit, must either avoid `State<AppState>` entirely or use optional state via `try_state` with a direct fallback. Add a regression check when changing these commands.
- When a release or CI run fails on only one platform, do not rerun or recreate completed artifacts unnecessarily. Inspect the failed job, fix the platform-specific cause, and preserve already-valid artifacts unless the release state itself is corrupt. If a release must be deleted and recreated, state why and re-verify all assets after recreation.
- A release is not complete until the currently installed app can update through the intended user path. If the app cannot open due to database schema skew, verify the database recovery updater path specifically, not only the normal About-page updater.
- If an in-app update path is broken in the currently installed release, use a reversible recovery bridge only with explicit backups: preserve the current database, temporarily restore a compatible backup if needed, perform the in-app update, then restore the preserved database and verify the final app/schema versions.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xiaoY233/cc-switch-remote](https://github.com/xiaoY233/cc-switch-remote) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
