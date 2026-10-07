---
trigger: always_on
description: The normal path is CI: push a `v*` tag, `.github/workflows/release.yml` builds
---

# CloudBridge — agent notes

## Local macOS release: build, sign, notarize, staple

The normal path is CI: push a `v*` tag, `.github/workflows/release.yml` builds
both platforms, signs and notarizes the macOS dmg, and publishes the GitHub
release. Reach for the local path below when CI is blocked rather than broken
— GitHub's macOS runner queue and Apple's notary queue have each stalled for
hours in practice, and neither is fixable from this repository.

`scripts/package-macos.sh` does the packaging itself and refuses to run
without credentials; the local path is the same script, run by hand, with no
job time limit.

### Prerequisites — already true on the maintainer's Mac

| What | Where |
|---|---|
| Signing identity | `Developer ID Application: Tian Deng (V5KP6ZYMDT)`, login keychain |
| Notary API key | `~/Downloads/AuthKey_492MX6SWPC.p8` |
| Key ID and Issuer ID | `scripts/notary.local` (gitignored, see below) |

Confirm the identity and its private key are present before anything else:

```bash
security find-identity -v -p codesigning | grep "Developer ID"
```

Exactly one identity must print. An `Apple Development` certificate cannot
sign for distribution. If the identity is missing, the certificate has to be
created in the Apple Developer portal — no script can do that step.

`scripts/notary.local` holds the three values the script needs:

```bash
export NOTARY_KEY=~/Downloads/AuthKey_492MX6SWPC.p8
export NOTARY_KEY_ID=492MX6SWPC
export NOTARY_ISSUER=<issuer UUID from App Store Connect -> Users and Access -> Integrations -> Keys>
```

The Key ID is not a secret (it is in the key's filename); the `.p8` is, and
stays where it is.

### Steps

1. **Check disk space.** Not only for a release: `target/` reaches 14 GB on
   an ordinary full build — 11 GB for the desktop debug build and its tests,
   3.3 GB for the wasm demo (measured 2026-09-24, with duckdb's bundled
   `json` and `parquet` extensions). A release build is on top of that.

   ```bash
   df -h /System/Volumes/Data   # want > 15Gi available
   ```

   9 GiB free is **not** enough: a cold `cargo test` from that gets most of
   the way through duckdb and then dies. The usual culprits are this and
   sibling Rust projects' `target/` directories — `cargo clean` in one of
   those is usually the fastest 20 GB you will find.

   On a full disk the failure is not a clean "out of space" —
   `libduckdb-sys` dies inside `ar cq` with no explanation beyond
   `errno=28`, and a build that gets further reports "failed to link or
   copy" on the final artifact. Both mean the disk, not the code.

2. **Build from a worktree of the tag, never from the working tree.** The
   maintainer usually has uncommitted work, and it must not end up in a
   release:

   ```bash
   git worktree add /tmp/cloudbridge-vX.Y.Z vX.Y.Z
   cd /tmp/cloudbridge-vX.Y.Z && cargo build --release
   ```

3. **Package with the script from `main`**, invoked by path so a tag that
   predates a packaging fix still gets the fix:

   ```bash
   source /Users/admin/side-proj/cloudbridge/scripts/notary.local
   cd /tmp/cloudbridge-vX.Y.Z
   /Users/admin/side-proj/cloudbridge/scripts/package-macos.sh \
     target/release/cloudbridge cloudbridge-macos-arm64.dmg X.Y.Z
   ```

   The script signs the app and the dmg, submits the dmg for notarization,
   waits, staples, then runs `stapler validate` and two `spctl` assessments.
   It must be run from the repository root — it reads `assets/` and `themes/`
   from the working directory.

   macOS will raise a GUI keychain prompt the first time `codesign` uses the
   key. Someone has to click **Always Allow**; an agent cannot.

4. **Expect to wait, and know that waiting is not failure.** Apple's notary
   service has taken anywhere from minutes to over 12 hours per submission
   for this team. Two facts follow:

   - The 5-hour `--timeout` in the script is a client-side wait, not a
     cancellation. When it expires the submission keeps processing on Apple's
     side, and the ticket can still be collected later.
   - **A ticket binds to the exact bytes submitted.** Do not rebuild after a
     timeout — a rebuild re-signs with a fresh timestamp, producing different
     bytes and no ticket. Staple the file that was submitted:

     ```bash
     xcrun notarytool history --key "$NOTARY_KEY" --key-id "$NOTARY_KEY_ID" --issuer "$NOTARY_ISSUER"
     xcrun notarytool info <submission-id> --key "$NOTARY_KEY" --key-id "$NOTARY_KEY_ID" --issuer "$NOTARY_ISSUER"
     # once the dmg's submission reads Accepted:
     xcrun stapler staple cloudbridge-macos-arm64.dmg
     ```

5. **Verify the artifact as a user's Mac would**, not just as the script
   does:

   ```bash
   xcrun stapler validate cloudbridge-macos-arm64.dmg
   spctl --assess --type open --context context:primary-signature --verbose=2 cloudbridge-macos-arm64.dmg
   MNT=$(hdiutil attach -nobrowse -readonly cloudbridge-macos-arm64.dmg | grep -o '/Volumes/.*' | head -1)
   spctl --assess --type execute --verbose=2 "$MNT/CloudBridge.app"
   hdiutil detach "$MNT" -quiet
   ```

   All four must report `accepted` with `source=Notarized Developer ID`.

6. **Publish.** Pushing a tag runs CI, which builds and publishes the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JetSquirrel/cloudbridge](https://github.com/JetSquirrel/cloudbridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
