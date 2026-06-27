---
trigger: always_on
description: - Review, approve, and manage open PRs for WinHome; enforce one-issue-per-contributor; ensure PRs are merged with `gssoc:approved` label.
---

# WinHome Session Progress

## Goal
- Review, approve, and manage open PRs for WinHome; enforce one-issue-per-contributor; ensure PRs are merged with `gssoc:approved` label.

## Constraints & Preferences
- `git pull` after every merge
- Prefer squash merges with descriptive subject lines; add `gssoc:approved` label on merge only (not on issues)
- Plugin files must end with POSIX trailing newline
- No `sys.exit(1)` on JSON parse error — return JSON error response
- All new plugins must include `requestId` in response dicts (JSON-RPC contract)
- `check_installed` must return bare `bool`; `main()` wraps into `{"requestId": ..., "installed": result}`
- `settings` from `args.get("settings", {})`, not `args` directly
- Test files use `sys.path.append` + `sys.path.remove` or `importlib`, not `sys.path.insert(0)`
- Dry-run returns `changed: True` when changes would be made
- Empty stdin must return JSON error response — silent return hangs host
- Atomic writes via `tempfile.mkstemp()` + `os.replace()`
- `requestId` uses `request.get("requestId") or "unknown"` — not `.get("requestId", "")`
- `dryRun` from `args`, not `context`
- `"data"` wrapper and `"success"` field banned from all responses
- PRs from non-assignees closed — verify assigned issues before review
- PRs must only touch files in scope of assigned issue
- Before accepting/assigning any plugin feature request or issue, verify `plugins/<name>/` doesn't already exist on `main` — if it does, close as duplicate
- Contributors with open PRs cannot receive new assignments
- Partial/split submissions for a single issue are not accepted — full feature must be delivered in one PR
- Repo homepage set to GitHub Pages: https://DotDev262.github.io/WinHome/

## Merged This Session
- **#385** (Deno, @silentguyracer) — Closes #330.
- **#428** (Multimedia docs, @Monica-CodingWorld) — Closes #400.
- **#415** (Module docs, @bhagya-2006) — Closes #236.
- **#430** (ScheduledTask identity bug, @ionfwsrijan) — Closes #425.
- **#423** (Package managers batch, @Stewartsson) — Closes #402.
- **#388** (Dual State Management, @ionfwsrijan) — Closes #376.
- **#384** (Docker multi-stage, @Stewartsson) — Closes #311.
- **#377** (Syncthing, @Bhagyashri77777) — Closes #181.
- **#371** (CliBuilder tests, @VIDYANKSHINI) — Closes #223.
- **#379** (Wallpaper Engine, @Stewartsson) — Closes #301.
- **#381** (Flow Launcher, @basantnema31) — Closes #141.
- **#422** (Spotify, @Devexhhh) — Closes #130.
- **#386** (Joplin, @Vidheendu) — Closes #184.
- **#433** (Greenshot, @Stewartsson) — Closes #293.
- **#439** (Bump JsonSchema.Net, @app/dependabot) — Dependency upgrade.
- **#432** (Sublime Text, @RaghuveerSingh05) — Closes #178.
- **#437** (Config Backup & Restore, @sat-06) — Closes #434.
- **#431** (Developer Tools Docs, @VIDYANKSHINI) — Closes #399.
- **#441** (dependsOn, @akshara200829-lgtm) — Closes #435.
- **#338** (NuGet, @lokeshkumar69) — Closes #328.
- **#442** (Postman, @Vidheendu) — Closes #291.
- **#417** (Fix Process PATH, @Bhagyashri77777) — Closes #392.
- **#457** (actions/checkout v6→v7, @app/dependabot) — Dependency bump.
- **#443** (Log file persistence, @Stewartsson) — Closes #147.
- **#460** (Spotify docs, @CH-GAGANRAJ) — Closes #453.
- **#462** (Joplin docs, @rj9884) — Closes #449.
- **#372** (VLC, @A-adilajaleel) — Closes #297.
- **#463** (Greenshot docs, @rj9884) — Closes #448.
- **#464** (Postman docs, @Stewartsson) — Closes #451.
- **#465** (Rainmeter docs, @Vishxlll20) — Closes #452.
- **#468** (Sublime Text docs, @hasitapattapu) — Closes #454.
- **#461** (Drift detection + source path metadata, @sat-06) — Full drift detection delivered. Closes #444, #425.
- **#458** (README entries, @Bhagyashri77777) — Closes #455.
- **#456** (Scoop docs rewrite, @Aashita101) — Closes #230.

## Closed This Session
- **#436** (README TOC, @Yogender-verma) — TOC redundant with DocFx site.
- **#427** (Logging - JSON, @Shashank1725) — Per author request. Closes #147 (unassigned).
- **#380** (Binary registry, @aayushprsingh) — Issue already fixed by #373. Non-assignee PR.
- **#383** (Plugin Health Checker, @Vidheendu) — Vague scope, core infra touches.
- **#382** (README TOC, @Yogender-verma) — Duplicate of #436 rationale.
- **#440** (Audacity duplicate, @ashu-here) — Plugin already on main via PR #354.
- **#474** (Landing page styling, @khushitripathi06) — Out of scope (CSS/design preference, DocFx styling).

## Unassigned
- **#236** (Docs, @priyanshi-coder-2) — 12 days, no PR.
- **#311** (Docker, @mahi-bansal) — 10 days, no PR.
- **#230** (Docs, @Stewartsson) — Per request (wanted #293).
- **#390** (Non-elevated shell, @Gnanesh67) — 9 days, no PR.

## New Assignments
- **#407** (Command Injection, @ionfwsrijan) — type:security, level:intermediate
- **#425** (ScheduledTask identity bug, @ionfwsrijan) — type:bug, level:beginner
- **#178** (Sublime Text, @RaghuveerSingh05) — type:feature, level:beginner
- **#293** (Greenshot, @Stewartsson) — type:feature, level:beginner
- **#434** (Backup & Restore, @sat-06) — type:feature, level:intermediate
- **#435** (dependsOn support, @akshara200829-lgtm) — type:feature, level:intermediate
- **#147** (Logging persistent, @Stewartsson) — type:feature, level:beginner

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DotDev262/WinHome](https://github.com/DotDev262/WinHome) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
