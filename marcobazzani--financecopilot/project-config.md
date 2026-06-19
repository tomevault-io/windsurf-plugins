---
trigger: always_on
description: - Version display: `v0.4.4` for stable releases, `v0.4.4-dev` for nightly/local builds. Controlled by `--dart-define=CHANNEL=stable|nightly` (defaults to `nightly`).
---

# Build & Deploy

- Version display: `v0.4.4` for stable releases, `v0.4.4-dev` for nightly/local builds. Controlled by `--dart-define=CHANNEL=stable|nightly` (defaults to `nightly`).
- When needed, always build first, then kill the running app, then start the new build. Never kill before the build completes.
  ```
  source .env && dart fix --apply && flutter build macos --release --dart-define=GOOGLE_CLIENT_ID=$GOOGLE_CLIENT_ID --dart-define=GOOGLE_CLIENT_SECRET=$GOOGLE_CLIENT_SECRET --dart-define=DB_FILE_NAME=$DB_FILE_NAME && pkill -f "FinanceCopilot" 2>/dev/null; open build/macos/Build/Products/Release/FinanceCopilot.app
  ```
- Android APK build:
  ```
  source .env && flutter build apk --release --dart-define=GOOGLE_CLIENT_ID=$GOOGLE_CLIENT_ID --dart-define=GOOGLE_CLIENT_SECRET=$GOOGLE_CLIENT_SECRET --dart-define=GOOGLE_WEB_CLIENT_ID=$GOOGLE_WEB_CLIENT_ID --dart-define=GOOGLE_ANDROID_CLIENT_ID=$GOOGLE_ANDROID_CLIENT_ID --dart-define=DB_FILE_NAME=$DB_FILE_NAME
  ```
- OAuth credentials are in `.env` (gitignored). Never commit secrets to git.

## Android Emulator

- Available emulators: `Medium_Phone_API_35`, `Pixel_8_Pro_API_35`
- Steps (in order):
  1. Launch emulator: `flutter emulators --launch <emulator_id>`
  2. Wait for it to appear: `flutter devices` (look for `emulator-XXXX`)
  3. Build APK: `source .env && flutter build apk --release --dart-define=GOOGLE_CLIENT_ID=$GOOGLE_CLIENT_ID --dart-define=GOOGLE_CLIENT_SECRET=$GOOGLE_CLIENT_SECRET --dart-define=GOOGLE_WEB_CLIENT_ID=$GOOGLE_WEB_CLIENT_ID --dart-define=GOOGLE_ANDROID_CLIENT_ID=$GOOGLE_ANDROID_CLIENT_ID --dart-define=DB_FILE_NAME=$DB_FILE_NAME`
  4. Install: `flutter install -d emulator-XXXX`
  5. Launch app: `adb -s emulator-XXXX shell monkey -p net.bazzani.financecopilot -c android.intent.category.LAUNCHER 1`
- Package name is `net.bazzani.financecopilot` (NOT `com.example.finance_copilot`).
- To run a second emulator alongside an existing one, just launch it — don't kill the first. They get sequential ports (5554, 5556, ...).
- If `am start` or `monkey` fails with "Activity does not exist" on a freshly launched emulator, the emulator image is likely corrupted (e.g. EdXposed or other framework mods). Fix: kill it (`adb -s emulator-XXXX emu kill`), relaunch with `flutter emulators --launch`, and reinstall.

## Windows VM (UTM)

The Windows build runs in a **UTM** virtual machine, controlled from the Mac with the bundled `utmctl` CLI. The commands below use placeholders — substitute your own values:

| Placeholder | Meaning | Example |
| --- | --- | --- |
| `<VM>` | UTM VM name (full name, or UUID from `utmctl list`) | `"Windows 11"` |
| `<USER>` | Windows username | `dev` |
| `<PROJECT>` | Project checkout path in the guest | `C:\Users\<USER>\dev\FinanceCopilot` |
| `<FLUTTER>` | Flutter `bin\flutter.bat` path in the guest | `C:\Users\<USER>\dev\flutter\bin\flutter.bat` |

- `utmctl` path (not on PATH): `/Applications/UTM.app/Contents/MacOS/utmctl`
- The Mac and the guest share the same `origin` git remote, so branches round-trip via GitHub. Find the guest IP with `utmctl ip-address "<VM>"` if you need direct network access.

### Critical `utmctl exec` caveats

- **No stdout/stderr is returned.** `utmctl exec` is fire-and-forget: it launches the process in the guest and returns immediately. It does NOT pipe back output and does NOT return the guest's real exit code (the host always sees exit 0 unless the *launch itself* fails, e.g. binary not found). To see ANY output you MUST redirect it to a file in the guest and pull it back with `utmctl file pull`.
- **Runs as `nt authority\system` (Session 0), not as `<USER>`.** This means: GUI apps must be launched via the scheduled-task trick (below); and `%APPDATA%`/`$env:APPDATA` resolve to SYSTEM's *own* profile, not the interactive user's — so reads that rely on those env vars come back empty. SYSTEM **can** read the interactive user's profile, but only via the **absolute** `C:\Users\<USER>\...` path. Write helper/log files to a SYSTEM-readable location such as `C:\` (the examples below use `C:\`).
- **Complex inline PowerShell gets corrupted.** `$variables` inside a `utmctl exec --cmd "powershell -Command \"...\""` are eaten by the host shell → quoting layers. For anything beyond a trivial one-liner, push a `.ps1` and run it with `--cmd "powershell.exe" "-NoProfile" "-ExecutionPolicy" "Bypass" "-File" "C:\script.ps1"`.
- **`file pull` races a still-open file.** While a command is writing to its redirect target, `file pull` fails with "The process cannot access the file because it is being used by another process." This is the completion signal's inverse: poll `file pull` and treat the lock error as "still running." Append a marker line (e.g. `DONE_EXITCODE=%ERRORLEVEL%`) as the last write so a successful pull containing the marker means "finished."

### Guest `.env`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [marcobazzani/FinanceCopilot](https://github.com/marcobazzani/FinanceCopilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
