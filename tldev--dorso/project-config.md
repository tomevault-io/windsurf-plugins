---
trigger: always_on
description: When the user says "ship it", perform the complete release workflow:
---

# Claude Code Instructions for Dorso

## Releasing

### Ship It (Full Release Workflow)

When the user says "ship it", perform the complete release workflow:

1. **Pull latest changes** with `git pull --rebase origin main` - ALWAYS do this first before committing
2. **Bump version** in `build.sh`
3. **Commit** cleanup/feature changes with proper attribution
4. **Merge** to main (if on a feature branch)
5. **Run `./release.sh X.Y.Z`** - builds, signs, notarizes, creates GitHub release
6. **Update release notes** with user-friendly description via `gh release edit`
7. **Update CHANGELOG.md** with new version entry
8. **Update README.md** contributors section if applicable
9. **Commit and push** changelog/readme updates
10. **Comment on PR/issue** thanking contributor and linking to release

### GitHub Release (Direct Distribution)

**IMPORTANT: Before releasing, always check existing releases and tags:**
```bash
gh release list --limit 5
git tag --sort=-v:refname | head -5
```
Only proceed if the version you're about to release doesn't already exist.

Always use the release script for GitHub releases:
```bash
./release.sh 1.0.X
```

This script handles:
- Building the app
- Code signing with Developer ID
- Apple notarization
- DMG creation with drag-to-Applications
- ZIP archive
- Git tagging
- GitHub release creation

**Never manually create GitHub releases** - the script ensures proper signing and notarization.

### After Running release.sh
The script creates generic release notes. **You must update them** with specific changes using:
```bash
gh release edit vX.Y.Z --notes "$(cat <<'EOF'
## What's New

### Bug Fix (or Feature, Improvement, etc.)
- **Short title** - Description of what changed and why it matters.

### Also in this release
- Any other notable points

## Installation

1. Download `Dorso-vX.Y.Z.dmg` or `Dorso-vX.Y.Z.zip`
2. Drag `Dorso.app` to Applications
3. Launch normally - no warnings!
4. Grant camera permission and complete calibration

## Requirements
- macOS 13.0 (Ventura) or later
EOF
)"
```

**Writing good release notes:**
- Keep it simple and user-focused - describe the benefit, not the implementation
- Avoid technical jargon (e.g., "brandCyan color" → "consistent styling")
- Don't repeat commit messages verbatim - synthesize changes into what users care about
- One clear sentence is better than a list of technical details
- Example: "Consistent styling across Settings and Analytics windows" instead of "Redesigned analytics window with brand-consistent styling, replaced shadows with borders, updated color scheme"

### Acknowledgments
When implementing features or fixes from GitHub issues, always give credit to the person who suggested it:
- In the GitHub release notes: "Thanks to @username for suggesting this!"
- In CHANGELOG.md: Add an `### Acknowledgments` section with a link to their GitHub profile
- In README.md: Add them to the Contributors section with a brief description of their contribution
- Comment on the issue thanking them and linking to the release

Note: Issues auto-close when referenced with "Closes #N" in commit messages. If you try to close an issue and it's already closed, just add a thank-you comment instead.

### Update CHANGELOG.md
After updating the GitHub release notes, also update `CHANGELOG.md` with a new entry:
- Add the new version section at the top (after the header)
- Use Keep a Changelog format with `### Added`, `### Changed`, `### Fixed` sections as appropriate
- Include the release date in YYYY-MM-DD format
- Commit and push the changelog update

### App Store Release
For App Store submissions, run these steps after the GitHub release:

```bash
cd /Users/tjohnell/projects/dorso

# 1. Build for App Store (excludes private APIs)
./build.sh --appstore

# 2. Copy to appstore folder and sign
rm -rf build-appstore/Dorso.app
cp -r build/Dorso.app build-appstore/
cd build-appstore

codesign --force --options runtime \
    --entitlements Dorso.entitlements \
    --sign "Apple Distribution: Thomas Johnell (KBF2YGT2KP)" \
    --timestamp \
    Dorso.app

# 3. Create installer package
rm -f Dorso.pkg
productbuild \
    --component Dorso.app /Applications \
    --sign "3rd Party Mac Developer Installer: Thomas Johnell (KBF2YGT2KP)" \
    Dorso.pkg

# 4. Upload (ask user for app-specific password)
xcrun altool --upload-app -f Dorso.pkg -t macos -u tjohnell@gmail.com -p APP_SPECIFIC_PASSWORD
```

**Important:** The upload requires an app-specific password from appleid.apple.com. Ask the user to provide it when uploading - do not store it in files.

After upload:
1. Go to App Store Connect → App Store tab
2. Select the version (e.g., "1.0 Prepare for Submission")
3. Scroll to Build section → click + → select the new build
4. Answer Export Compliance: "No" (no encryption)
5. Save → Add for Review → Submit to App Review

## Build Configurations

- `./build.sh` - Regular build with private APIs (for GitHub/direct distribution)
- `./build.sh --appstore` - App Store build without private APIs
- `./build.sh --release` - Regular build + creates ZIP archive

## Installing During Development

**Always kill the existing process and remove old app before installing:**
```bash
pkill -x Dorso; rm -rf /Applications/Dorso.app && cp -r build/Dorso.app /Applications/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tldev/dorso](https://github.com/tldev/dorso) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
