---
trigger: always_on
description: |
---


# Qiaomu Suno Master

Create commercial-grade Suno lyrics, then use the selected Suno execution lane
to generate and optionally download the music.
It also includes a local music genre finder so vague moods can become precise Suno style tags.

Suno login state is short-lived. Treat Chrome's logged-in Suno web session as
the source of truth, and treat the CLI as the fast path only. If the CLI reports
`auth_expired`, `JWT expired`, `401`, `403`, captcha failures, or cannot find a
browser session, immediately use the Suno web UI fallback instead of retrying the
same CLI request.

## Style Tag Selection Contract

When the user asks for Suno style tags, genre combinations, genre discovery, or
style sharpening, treat the local genre finder as mandatory evidence, not an
optional helper.

1. Read `references/genre-selection.md` before answering.
2. Run `python3 scripts/find_music_genres.py` from the skill directory. For
   style-only requests or broad briefs, run at least three focused queries that
   sample different axes of the brief: the user's original phrase, one or more
   adjacent genre families, and one or more production/texture/rhythm angles.
   A single query is acceptable only when the user gives a very narrow style and
   asks for minimal output.
3. Build a candidate palette from the database results first. Prefer genre tags
   that appeared in the finder output or are direct, obvious parents/children of
   those results.
4. Compose each Suno style string as 1-3 genre tags plus 2-5 vocal,
   instrument, tempo, production, or mood tags. Do not answer from taste alone,
   and do not dump a long related-genre list into one prompt.
5. For style-only requests, do not write lyrics or run Suno generation unless
   the user explicitly asks. Return useful combinations, the best starters, and
   `exclude_styles` when helpful.
6. If the finder fails or the database is unavailable, say that clearly and
   continue with a best-effort fallback instead of implying the tags were
   database-backed.

## Generation Execution Contract

This skill must prefer a deterministic lane over open-ended exploration.

0. **Honor explicit lane requests as a hard lock.**
   - If the user explicitly asks for Computer Use, the whole generation task
     stays on the Computer Use lane from form submission through visible link
     capture and browser download attempts. Do not switch to CLI, CDP,
     Browser plugin, raw API calls, or captcha-assisted CLI generation to
     "make progress".
   - Treat "全程用 Computer Use", "直接用 Computer use", "用电脑操作", and
     follow-up corrections about route mixing as a Computer Use hard lock.
     Once this lock is active, every Suno-facing action for that request must
     happen through the visible Suno UI controlled by Computer Use.
   - In a Computer Use-locked task, shell commands may prepare lyrics,
     manifests, inspect files, move browser-downloaded files, resolve captured
     share links, and validate assets. They must not submit Suno generation,
     refresh generation auth, run `suno generate`, or drive CDP/CLI captcha
     helpers.
   - In a Computer Use-locked task, do not run CDP preflights, launch a
     CDP-enabled Chrome, call the Browser plugin, or use `download_clips.sh`
     as a hidden fallback. If visible UI download fails, report the blocker
     with the captured links instead of switching lanes.
   - If Computer Use cannot proceed because the user is actively using the same
     browser, the page requires login/security/captcha action, or the desktop UI
     is unavailable, stop at that state and ask for that blocker to be cleared.
     Do not silently fall back to another generation lane.
   - The same lock-in rule applies to any other lane the user explicitly names:
     use that lane end to end, or stop and report the blocker.

1. **Make the CLI path work first.**
   Use this only when the user did not explicitly request another lane.
   - Ensure the installed `suno` CLI exists and run `suno config check`.
   - Refresh auth from the real Chrome Suno session; if refresh fails, run
     `suno auth --login --quiet`.
   - Run `scripts/generate_with_suno.sh` once with the default captcha-backed
     path.
   - Retry CLI at most one more time only for the narrow hCaptcha/CDP-launch
     failure class, using either `--no-captcha` or a user-provided
     `--token "$HCAPTCHA_TOKEN"`.
2. **If CLI generation is blocked, Codex controls the browser.**
   Use this only when the task is not locked to a different lane.
   - Use the Codex Browser plugin when available. If it is not exposed in the
     current session, use Chrome/Computer Use against the logged-in Suno page.
   - Open `https://suno.com/create`, fill title, lyrics, styles, model, and
     options from the prepared local files, click Create, wait for the generated
     rows, and capture song IDs/links.
   - Only ask the user to intervene for actions automation cannot legally or
     reliably complete, such as human login, account security checks, or a live
     captcha challenge.
3. **Stop forbidden exploration.**
   - Do not hand-craft Suno generate POST requests, inject copied browser cookies
     into throwaway profiles, replay captured payloads, or repeatedly test
     captcha variants unless the user explicitly asks to debug Suno itself.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joeseesun/qiaomu-suno-master](https://github.com/joeseesun/qiaomu-suno-master) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
