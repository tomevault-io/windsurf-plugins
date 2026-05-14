---
trigger: always_on
description: Project-specific guardrails. **Read every section before touching this repo.**
---

# CLAUDE.md

Project-specific guardrails. **Read every section before touching this repo.**

## Release process — MANDATORY

This app ships via Sparkle auto-update. Get any of this wrong and you brick auto-update for everyone who's already installed.

### The loop (3 commands)

```sh
echo "X.Y.Z" > VERSION                                # 1. Bump
git commit -am "chore(release): bump VERSION to X.Y.Z" \
  && git tag vX.Y.Z                                   # 2. Commit + tag
git push origin main vX.Y.Z                           # 3. Push (fires CI)
```

The marketing landing site at `ericjypark/codex-island-landing` has its own
`VERSION` file (the hero chip + footer read it at build time). Bump it in
that repo too, in the same release sweep, or the public site keeps showing
the prior version even after `brew install` ships the new one.

That's it. CI does **everything else** in ~1.5 min:

- Builds the universal DMG
- Signs it with the EdDSA key from the `SPARKLE_ED_PRIVATE_KEY` secret
- Generates `appcast.xml` listing the new version
- Uploads DMG + appcast as release assets
- Mirrors the cask to `ericjypark/homebrew-tap` with the new version + SHA-256

Watch with `gh run watch --exit-status` if you want confirmation, or just trust it.

### Hard rules — break these and you brick auto-update

1. **`VERSION` must be a single-monotonic version like `0.0.X`, NOT `1` or `100` or anything weird.** `build.sh` uses `$VERSION` as both `CFBundleVersion` and `CFBundleShortVersionString`. Sparkle compares `CFBundleVersion` of the running app against `sparkle:version` in the appcast using Apple's component-wise comparator — so `"1"` parses as `[1]` and is **larger than** `"0.0.99"`. Stay in semver. Always increase.

2. **The Sparkle public key in `build.sh` (`SU_PUBLIC_KEY="bz1g..."`) must NEVER be changed casually.** Every existing install verifies updates against this exact key. Change it and every prior install rejects every future update silently. The matching private key lives in (a) the maintainer's macOS Keychain under service `https://sparkle-project.org` and (b) the `SPARKLE_ED_PRIVATE_KEY` GitHub Actions secret. To rotate, see the migration note in `docs/SPARKLE.md` (TL;DR: don't).

3. **Don't manually edit `Casks/codexisland.rb` for a version bump.** CI rewrites it on the homebrew-tap side at release time. Manual version/SHA edits are overwritten or drift. (Editing unrelated cask metadata — postflight, zap, livecheck — via a normal commit is fine; CI preserves those.)

4. **Never edit appcast XML files by hand.** The appcast is a release asset built by `release.sh` from the signed DMG. Hand-edits invalidate the EdDSA signature.

5. **Never commit `Vendor/`.** It's gitignored. The `bin/sign_update`, `bin/generate_keys`, etc. binaries live there for local use; CI re-vendors via `scripts/setup-sparkle.sh`.

### CI secrets (one-time, already configured)

These two GitHub Actions secrets exist on the `codex-island` repo:

- **`SPARKLE_ED_PRIVATE_KEY`** — the EdDSA private key. Without it CI fails at the signing step.
- **`HOMEBREW_TAP_TOKEN`** — fine-grained PAT with `contents: write` on `ericjypark/homebrew-tap` only. Without it the cask-sync step warns and skips, but the GitHub Release still ships.

If either is rotated, regenerate via the original instructions in `docs/SPARKLE.md`.

### Smoke-testing the update prompt locally

If you want to verify Sparkle's UI before tagging:

```sh
./release.sh                  # produces dist/CodexIsland-X.Y.Z.dmg + dist/appcast.xml
                              # (uses Keychain key — no env vars needed locally)
```

The local `release.sh` is identical to CI except for asset upload. To force-trigger an update prompt without publishing: temporarily change `SUFeedURL` in `build.sh` to point at `http://127.0.0.1:8765/appcast.xml`, serve `dist/appcast.xml` from there with `python3 -m http.server 8765`, run with a lower local `VERSION` than the appcast advertises, hit Check Now.

To build with auto-update **disabled** (debug copies): `SU_FEED_URL= ./build.sh`.

### Things that have already broken and how they were fixed

History — read before re-stepping on these rakes:

| Problem | Symptom | Root cause | Fix |
|---|---|---|---|
| `CFBundleVersion = "1"` hardcoded | Sparkle never sees any update as newer | `"1"` > `"0.0.X"` in component-wise comparison | `build.sh` now sets it to `$VERSION` |
| `SUPublicEDKey` empty in CI builds | Sparkle silently rejects every signed update | Public key was in gitignored `Vendor/` only | Public key hardcoded in `build.sh` |
| `xattr -d` non-recursive in cask postflight | "Updater failed to start" on first Check Now | Quarantine attr remained on Sparkle's nested `Updater.app` | `xattr -dr` (recursive) |
| `--no-quarantine` in install docs | `brew install` fails with "switch is disabled" | Homebrew removed the flag in late 2025 | Cask postflight strips the attr; flag removed from docs |
| `…` after `$VAR` in shell scripts | CI fails with `unbound variable` | Non-UTF-8 locale on runners makes bash include trailing bytes in identifier | Use `${VAR}…` braces, or stick to ASCII in echo strings |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ericjypark/codex-island](https://github.com/ericjypark/codex-island) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
