---
trigger: always_on
description: Lyrenne is a YouTube Music player for Windows, written in Kotlin on Compose Desktop (JVM). It is
---

# Working on Lyrenne as an AI agent

Lyrenne is a YouTube Music player for Windows, written in Kotlin on Compose Desktop (JVM). It is
**not** an Android app. Any documentation describing this repo as an Android project with Hilt,
Room and Media3 is stale and describes the upstream project it began as.

`CLAUDE.md` is the detailed project guide. This file is the short version plus the traps.

## What this actually is

- **Target**: Windows 10 and 11 desktop, JVM 21. There is no mobile build.
- **Modules**: only `:desktop` is a Gradle project. The `innertube`, `lrclib`, `betterlyrics`,
  `kugou`, `lastfm` and `shazamkit` folders are Android library modules whose *sources* are pulled
  in via `kotlin.srcDir()`, because a JVM target cannot consume Android libraries as project
  dependencies. `app/` is the original Android app, kept only as a porting reference, never built.
- **Stack**: Compose Desktop UI, VLC through vlcj for playback, SQLDelight for the database,
  singleton managers instead of dependency injection.
- **Package namespace** is `com.lyrenne.desktop`. The `com.metrolist.*` imports you will still
  see are the vendored upstream modules (innertube, lrclib, kugou, lastfm, shazamkit,
  betterlyrics). Those are someone else's GPL sources consumed via `kotlin.srcDir()`: renaming
  them would conflict with every future upstream sync, so leave them alone.

## Build and test

```bash
./gradlew :desktop:compileKotlin      # fast check
./gradlew :desktop:createDistributable # runnable app
./gradlew :desktop:packagePortableZip  # release archive
```

There is no APK, no emulator and no `:app:assembleFossDebug`.

## Rules that are not style preferences

Each of these exists because breaking it caused real damage.

1. **Never run the app from `desktop/build/compose/binaries/main/app/Lyrenne/`.** `AppPaths` writes
   `data/` next to the executable, so running it there plants real login cookies in the exact
   folder that gets zipped for release. This shipped publicly once. Test from a copy outside the
   build tree.
2. **Never hand-roll the release ZIP.** `packagePortableZip` purges runtime dirs and then fails the
   build if the archive contains credentials, the database, preferences or a log.
3. **Never use PowerShell `Compress-Archive`.** It writes backslash entry names, which breaks
   Java's `ZipEntry.isDirectory()` and therefore the auto-updater. Use 7-Zip.
4. **Do not encrypt or obfuscate `credentials.json`.** Plaintext is intentional and required.
5. **Renaming a user data path orphans every existing install.** 2.9.4 did exactly that on
   purpose, trading upgrade smoothness for a clean break, and shipped manual recovery steps in the
   release notes. Do not do it again casually: if a path must change, ship a migration.
6. **Discord Rich Presence must never re-send on position ticks.** An earlier version did, which
   raced the named pipe and hit Discord's rate limit. It now sends on song change, a detected
   seek, a duration correction from VLC, and pause or resume. Every one of those is a discrete
   event that fires about once per track, and the seek and pause paths are both debounced. Any
   new trigger has to clear the same bar: discrete, roughly once per track, debounced.
7. **Do not bump the version** unless asked. It lives in two places that must match:
   `desktop/build.gradle.kts` and `AutoUpdater.CURRENT_VERSION`.
8. **`data/login-profile` must never outlive a sign-in.** It holds a live Google session. Deleting
   only `credentials.json` on logout left that session on disk, so signing out was not signing out.
   Delete it on successful extraction, on logout, and at startup. Only on `Success` though: the
   handoff path re-reads the profile while the user is still signing in.
9. **Never let the updater off HTTPS or off GitHub.** `requireTrustedUrl()` gates the download URL
   and every redirect hop. What it fetches is executed, and nothing is signed.
10. **Paths interpolated into the generated PowerShell update script go through `psQuote()`.**
    `$` and `$(...)` are both legal in Windows folder names, so double quotes there mean silent
    wrong-directory copies at best and execution at worst.
11. **Do not blanket-add `key =` to lazy lists.** Queue and local playlists can hold the same song
    twice, and Compose throws on duplicate keys. Only collections with database primary keys are
    safe, which is the Library lists and nothing else.
12. **No em dashes.** Not in UI strings, docs, comments or commit messages. The project owner does
    not write with them and will ask for them to be removed.
13. **Never read `SongInfo.durationMs` or `.duration` directly.** They are the same fact in
    different units and only one is ever populated, depending on where the song came from. Use
    `knownDurationMs()`, and prefer the live `PlaybackState.duration` when it is non-zero.
14. **Clear `isSyncing` in a `finally`, and never suspend there outside `NonCancellable`.** A
    suspending call in a `finally` throws the moment the job is cancelled, so the reset after it
    never runs. Leaking the flag silently stops this client sending anything to the room.

## Things that look broken but are not


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Doctordefector/Lyrenne](https://github.com/Doctordefector/Lyrenne) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
