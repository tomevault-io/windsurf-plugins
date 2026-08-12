---
trigger: always_on
description: Most fixes only touch daemon TypeScript. No Rust re-compile needed.
---

# Sentinel – Development Guide

## Fast iteration loop (daemon changes only)

Most fixes only touch daemon TypeScript. No Rust re-compile needed.

**IMPORTANT: Never run `pkill`/`kill` on the daemon while the user has the Tauri app open.**
The Tauri app spawns the daemon once on startup and does NOT auto-restart it if it exits.
Killing it breaks the user's session with no automatic recovery.

The safe way to deploy new daemon code:

```sh
# 1. Build
pnpm --filter @sentinel/daemon run build          # tsc
pnpm --filter @sentinel/daemon run build:sidecar  # pkg → binary

# 2. Replace the binary in the installed app bundle (safe while the daemon is running —
#    the running process keeps its inode; the new binary takes effect on next launch)
cp "packages/app/src-tauri/binaries/sentinel-daemon-aarch64-apple-darwin" \
   "/Applications/Sentinel.app/Contents/MacOS/sentinel-daemon"
echo "Binary replaced — ask the user to restart Sentinel."

# 3. Tail logs after the user restarts
tail -f ~/.sentinel/daemon.log
```

## Test cycle checklist

Before declaring a fix complete, always:

1. Build daemon (if changed): `pnpm --filter @sentinel/daemon run build && pnpm --filter @sentinel/daemon run build:sidecar`
2. Build full app (if frontend/Rust changed): `pnpm --filter @sentinel/app run tauri:build`
3. Ask the user to quit Sentinel, then install + open
4. **Monitor logs throughout**: `tail -f ~/.sentinel/daemon.log`
5. Confirm expected log lines appear (daemon start, IPC responses, OAuth flow, broadcasts)

## Testing without disturbing the live daemon

Use the IPC script to send messages to the RUNNING daemon:

```sh
# List accounts
node scripts/ipc.mjs '{"type":"refresh_accounts"}'

# Switch account (supply real UUID from the list above)
node scripts/ipc.mjs '{"type":"switch_account","accountId":"<uuid>","email":"<email>"}'
```

Verify a switch by reading the results directly:

```sh
# Active account in ~/.claude.json
node -e "const s=require('fs').readFileSync(require('path').join(require('os').homedir(),'.claude.json'),'utf-8'); const p=JSON.parse(s); console.log(p.oauthAccount?.emailAddress, p.oauthAccount?.accountUuid)"

# Token in Claude Code's keychain (macOS) — confirm it changed
security find-generic-password -s "Claude Code-credentials" -a "$USER" -w 2>/dev/null \
  | node -e "const d=require('fs').readFileSync('/dev/stdin','utf-8').trim(); const p=JSON.parse(d); console.log('token prefix:', p.claudeAiOauth?.accessToken?.slice(0,30))"

# Daemon health
curl -s http://localhost:47284/health
```

Test the profile API directly with an existing access token:

```sh
AT=$(security find-generic-password -s "Sentinel-credentials" -a "<key>" -w 2>/dev/null \
     | node -e "process.stdout.write(JSON.parse(require('fs').readFileSync('/dev/stdin','utf-8').trim()).accessToken||'')")
node -e "
(async () => {
  const r = await fetch('https://api.anthropic.com/api/oauth/profile', {
    headers: { Authorization: 'Bearer ${AT}', 'Content-Type': 'application/json' }
  });
  const d = await r.json();
  console.log(JSON.stringify({ uuid: d.account?.uuid, email: d.account?.email, orgType: d.organization?.organization_type }, null, 2));
})();
"
```

## Full app build (Rust/frontend changes)

```sh
pnpm build:app
```

`pnpm build:app` is the **cross-platform** dev entrypoint
(`scripts/build-app.mjs`): it detects the OS, builds **unsigned** (no
`~/.tauri/*.key` prompt, via `src-tauri/tauri.dev.conf.json` which sets
`bundle.createUpdaterArtifacts: false`), then launches your changes. macOS
installs to `/Applications`, ad-hoc re-signs, and opens; Linux builds and
runs an `.AppImage`; Windows builds and runs the NSIS `-setup.exe`. No
Apple Developer ID signing happens unless `APPLE_SIGNING_IDENTITY` is
exported. Preview without building: `node scripts/build-app.mjs --dry-run`
(add `--platform=linux|win32|darwin`). `install:app` is a back-compat
alias for the same script.

Use `pnpm build:app:release` for the SIGNED release build (full
`tauri build`, all targets, signed updater artifacts — prompts for the key
password). CI does this via tauri-action; you rarely need it locally. Do
NOT use `build:app:release` for the dev loop — it blocks on the key prompt.

On macOS the dispatcher delegates to `scripts/install-app.sh`, which builds
the unsigned `.app`, replaces `/Applications/Sentinel.app` cleanly
(`rm -rf` first to avoid the `cp -R` merge-not-replace footgun), re-signs
ad-hoc, verifies, and launches. It aborts if Sentinel is still running —
quit it from the tray first.

Never `cp -R` a build over the existing app manually: macOS protects the
installed bundle's files, so each `Contents/...` file fails with
"Operation not permitted". The `rm -rf`-first swap is what avoids it. The
re-sign step is also mandatory: `cp -R` over an existing bundle leaves
macOS's amfid/Gatekeeper cache stale, and the first child the app spawns
(the daemon sidecar) gets SIGKILLed silently — looks identical to "the
daemon didn't open". `codesign --force --deep --sign -` clears the cache.
The script handles all of this; do not skip it by running steps manually.

Artifacts also land in `packages/app/src-tauri/target/release/bundle/macos/`
for direct inspection of the .app (the dev build skips the `.dmg`).

## Tests

### Running tests

```sh

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Intevity/sentinel](https://github.com/Intevity/sentinel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
