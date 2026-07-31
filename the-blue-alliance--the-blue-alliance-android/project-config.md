---
trigger: always_on
description: The TBA web server (backend + frontend) is checked out at `~/codez/the-blue-alliance-2026` with its Docker container running. When the app targets the local dev server (e.g. `http://10.0.2.2:8080` in the emulator), requests go to that container.
---

# Local Development Notes

## Local Backend

The TBA web server (backend + frontend) is checked out at `~/codez/the-blue-alliance-2026` with its Docker container running. When the app targets the local dev server (e.g. `http://10.0.2.2:8080` in the emulator), requests go to that container.

## Testing Signed-In Features Locally

Signed-in features (myTBA favorites/notifications — anything gated on `isSignedIn` / Firebase
Auth) are testable locally **without a real Google account**. Debug builds wire Firebase Auth to
the local emulator: `AuthModule` calls `useEmulator("10.0.2.2", 9099)` whenever `BuildConfig.DEBUG`.

1. Make sure the local stack is up — the Docker Compose backend includes the Firebase Auth
   emulator on `:9099`. Check: `curl -s localhost:9099` returns `200`.
2. Install + launch via the script — `scripts/install-and-launch.sh`, or `scripts/emu launch <component>`
   after a build. This grants Android 17's `ACCESS_LOCAL_NETWORK`; **without it the app can't reach
   `10.0.2.2`** (the backend *or* the auth emulator), so sign-in silently fails.
3. In the app, tap **Sign in with Google**. In debug this calls `MainActivity.signInWithEmulator()`,
   which signs in a fake user (`user@thebluealliance.com`) against the emulator — no OAuth dialog.

`isSignedIn` is then true and the myTBA tabs render (otherwise the screen shows only the sign-in
prompt). For myTBA data to sync, point `tba.url.debug` at the local backend (`http://10.0.2.2:8080/`).

## Build Commands

```bash
./gradlew :app:assembleDebug
./gradlew :app:installDebug   # always install to emulator after building
./gradlew :app:testDebugUnitTest
```

After installing a debug build, always relaunch the app on the emulator with `scripts/emu launch`
(it also grants Android 17's `ACCESS_LOCAL_NETWORK` so the `10.0.2.2` local backend stays reachable):
```bash
scripts/emu launch com.thebluealliance.androidclient.development/com.thebluealliance.android.MainActivity
```

## Emulator Interaction

Use `scripts/emu` (a Python CLI included in the repo) instead of raw `adb` commands. It provides text-based UI element matching which is far more reliable than guessing pixel coordinates.

```bash
scripts/emu screenshot screenshots/<name>.png   # capture screenshot
scripts/emu find "text"                          # find UI elements by text
scripts/emu tap "text"                           # tap element by text (must match exactly one)
scripts/emu tap-xy <x> <y>                       # tap at exact device pixel coordinates
scripts/emu back                                 # press BACK key
scripts/emu list                                 # dump full UI hierarchy as readable tree
scripts/emu logcat --tag <tag> --grep <pattern> -n <count>  # filtered logcat
scripts/emu launch <package/activity>            # force-stop and start activity
```

Use `find` before `tap` to verify unique matching; use `list` to explore the UI hierarchy.

## PR Screenshots (before/after in PR descriptions)

This is the standard way to put before/after screenshots in a PR — WITHOUT committing
image files. Images live on an orphan `screenshot-assets` branch (its own history, never
merged) and serve from `raw.githubusercontent.com`, so they render inline in the PR body.
Release assets do NOT work here — the org has immutable releases enabled.

```bash
# Uploads images, then rewrites the PR body's screenshot block in place.
scripts/pr-screenshots.sh --pr <N> \
  --row "Label" before.png after.png \   # a before/after table row (repeatable)
  --shot "Label" single.png              # a single image (repeatable)
```

Omit `--pr` to just print the markdown. Re-runs cleanly replace the
`<!-- screenshots:start/end -->` block. Works on fork PRs too (run as a maintainer).
The script never touches your working tree, index, or current branch (pure git plumbing
into the asset branch). Each filename is content-hashed so GitHub's image proxy can't
serve a stale cached copy.

**Getting real data for the shots.** The local backend has no competition data, so point
the *debug* build at prod with a personal read key (thebluealliance.com/account) in
`local.properties`, then rebuild:
```
tba.url.debug=https://www.thebluealliance.com/
tba.api.key.debug=<your-read-key>
```
Drive navigation with deep links — more reliable than tapping, and in-app search only
matches teams/events you've already browsed:
```bash
adb shell am start -n com.thebluealliance.androidclient.development/com.thebluealliance.android.MainActivity \
  -a android.intent.action.VIEW -d "https://www.thebluealliance.com/event/2024micmp4"
```

**Flushing the asset branch when it gets too full.** Assets accumulate on the branch
forever. To reset it (the script recreates it on the next run):
```bash
git push origin --delete screenshot-assets
```
This breaks image links in any *already-merged/closed* PRs that referenced old assets, so
only flush when those are done. Open PRs you still care about should be re-screenshotted
after a flush.

**Heads-up:** the Docker backend + a Gradle build + the emulator running together can OOM

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [the-blue-alliance/the-blue-alliance-android](https://github.com/the-blue-alliance/the-blue-alliance-android) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
