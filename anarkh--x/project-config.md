---
trigger: always_on
description: Guidance for AI coding agents working on this repository.
---

# AGENTS.md

Guidance for AI coding agents working on this repository.

## Project Overview

Street Tasks is a native WeChat mini program for short-lived neighborhood tasks. The current product is intentionally small: a map-first feed, a publish flow, a task detail page, comments, structured trust actions, lightweight admin moderation, profile/activity surfaces, and feedback. There is no fixed service area; users can browse and publish from any current location.

The app can run locally from mock data and `wx` local storage, and it also has CloudBase-backed paths for shared posts, reactions, comments, feedback, viral attribution events, images, and admin role checks. Treat `utils/store.js` as the main persistence boundary; page code should not duplicate storage or cloud fallback logic.

## Harness Operating Loop

This repository keeps agent state, verification, and handoff files under `harness/`. Treat those files as the durable source of truth for long-running AI-assisted work.

Before changing code:

1. Run `pwd` and confirm the repo root is `/Users/bytedance/git/x`.
2. Read `harness/claude-progress.md` for the latest verified state, blocker, and next action.
3. Read `harness/feature_list.json` and pick the highest-priority unfinished feature. Keep at most one feature `in_progress`.
4. Check recent history with `git log --oneline -5`.
5. Run `bash harness/init.sh`. If it fails, fix the base state before adding feature work.

Required harness files:

- `harness/feature_list.json`: machine-readable feature status and verification evidence.
- `harness/claude-progress.md`: session log and current verified state.
- `harness/init.sh`: single bootstrap and baseline verification entrypoint.
- `harness/session-handoff.md`: short handoff summary for longer sessions.
- `harness/clean-state-checklist.md`: closeout checklist before handing work back.
- `harness/evaluator-rubric.md`: acceptance rubric for completed work.
- `harness/quality-document.md`: project quality snapshot by product area and architecture layer.

Completion definition:

- The target behavior is implemented.
- Required verification actually ran.
- Evidence is recorded in `harness/feature_list.json` or `harness/claude-progress.md`.
- The repo can still be restarted from `bash harness/init.sh`.
- Any skipped manual WeChat DevTools checks are called out as unverified, not implied passing.

Session closeout:

1. Update `harness/claude-progress.md`.
2. Update `harness/feature_list.json` when feature status or evidence changes.
3. Note unresolved risks or blockers.
4. Run the relevant verification commands.
5. Leave the next session able to continue from repo files alone.

## Tech Stack

- Native WeChat mini program files: `.js`, `.json`, `.wxml`, `.wxss`
- JavaScript ES modules
- Local persistence through `wx.getStorageSync` / `wx.setStorageSync`
- WeChat location APIs using `gcj02`
- No build framework and no frontend package bundler

## Important Files

- `app.js`: app bootstrap, guest user initialization, and global center state.
- `app.json`: page registry, tab bar, window style, and location permission declaration.
- `project.config.json`: public WeChat DevTools config. Keep `appid` as `touristappid` for GitHub.
- `project.private.config.json`: local-only WeChat DevTools config. This may contain the real AppID and must stay ignored.
- `utils/config.js`: nearby feed config, categories, and expiry options.
- `utils/store.js`: post, comment, reaction, image upload, local storage, and CloudBase fallback APIs.
- `utils/viral-attribution.js`: best-effort share landing, conversion, and relay attribution events with local fallback and a strict field whitelist.
- `utils/auth.js`: local user, profile completion, admin role refresh, and permission helpers.
- `utils/feedback.js`: user feedback creation and admin feedback listing.
- `utils/geo.js`: distance calculation and map marker conversion.
- `utils/format.js`: category and time display helpers.
- `utils/post-presenter.js`: shared presentation helpers for profile/activity pages.
- `utils/diagnostics.js`: runtime diagnostics used by map startup and fallback paths.
- `utils/mock-posts.js`: seed data used when local storage is empty.
- `harness/*`: agent harness state, verification, closeout, and quality tracking files.
- `DESIGN_SYSTEM.md`: current visual design rules and native/TDesign-style component patterns.
- `PROJECT_SUMMARY.md`: high-level project summary for humans and future agents.
- `pages/map/*`: map feed, marker interactions, and list overlay.
- `pages/publish/*`: task creation flow.
- `pages/detail/*`: detail view, images, comments, confirm/stale/report actions, and resolve flow.
- `pages/admin/*`: admin-lite moderation view.
- `pages/me/*`: login, profile, admin entry, and personal stats.
- `pages/my-posts/*`: current user's posts.
- `pages/activities/*`: current user's trust-action history.
- `pages/feedback/*`: user feedback form.
- `cloudfunctions/posts/index.js`: CloudBase actions for posts, reactions, comments, feedback, viral attribution events, and image upload preparation.
- `cloudfunctions/getMyRole/index.js`: CloudBase admin role lookup from the `admins` collection.
- `scripts/check-json.mjs`: JSON syntax check for project and page config files.

## Local Development


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anarkh/x](https://github.com/anarkh/x) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
