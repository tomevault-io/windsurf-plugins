---
trigger: always_on
description: These rules are project-scoped and override anything that conflicts in
---

# Tideway — project instructions

These rules are project-scoped and override anything that conflicts in
the global Claude config. Global rules about writing style, commit
attribution, and git hygiene still apply.

## No bandaids

If you recognize that what you're about to write is a workaround for
a deeper problem rather than the actual fix, **stop and find the
actual fix.** Things that are bandaids:

- "Belt-and-suspenders" / "defense-in-depth" setState calls that
  paper over a state-management bug whose root cause is somewhere
  else. The right fix is in the somewhere-else.
- Hardcoded substring / name-matching filters for things the OS
  exposes through a proper API (CoreAudio's
  `kAudioDevicePropertyDeviceCanBeDefaultDevice`, WASAPI's
  `IMMDevice::GetState()`, etc). The right fix is the OS API.
- "Sometimes one isn't enough so we do it twice" loops, retry
  counts above 1 without an explicit reason, sleeps that exist
  because something is racy. The right fix is to find what's racy
  and synchronize properly.
- Try / except blocks that catch a real error and turn it into a
  silent fallback. Catch a specific error class with a comment
  explaining why; don't swallow `Exception` and pretend everything
  is fine.
- Comments that say "for now" or "until we" or "as a workaround"
  on permanent code paths. If it's not the right fix, don't ship
  it; if it IS the right fix, the comment shouldn't apologize for
  it.

When the proper fix is genuinely outside the scope of the current
change (e.g. needs a different platform's API and you're not on
that platform), say so explicitly and either:

1. Punt to a follow-up branch with a clear scope, or
2. Leave the un-fixed behavior intact (degraded but honest) rather
   than ship a misleading bandaid that masks the gap.

Never ship code that you'd describe to the reviewer as "temporary"
or "good enough for now" without explicit user sign-off on that
specific tradeoff.

## Release workflow

When the user has a batch of unrelated fixes or features to ship in a
single deploy, the workflow is:

1. **One branch per fix.** Branch off `main`. Naming: `fix/<slug>` for
   bug fixes, `feature/<slug>` for new capability, `chore/<slug>` for
   refactors / tooling. One concern per branch — if the work touches
   two unrelated subsystems, split it.
2. **One PR per branch.** PR targets `main`. Title is what the work
   does, not a position in a queue ("Pause queue advance on track-
   end" beats "Bug 3 of 7"). Body has Summary + Test plan.
3. **Hold approved PRs.** Each PR gets reviewed and approved, but is
   NOT merged on its own. The approval is "this fix is ready", not
   "ship it now."
4. **Integrate on a deploy branch.** When the batch is ready to ship:
   - Create `deploy/v<X.Y.Z>` off the latest `main`.
   - Merge each approved branch into the deploy branch with
     `git merge --no-ff <branch>` so the integration history is
     preserved.
   - Resolve any conflicts here, in the integration branch. The
     individual PR branches stay clean.
   - Add a single release commit on top of the deploy branch:
     - Bump version in any version-pinned file.
     - Write release notes (one paragraph per included PR, in past-
       tense human language).
5. **Test the integrated deploy branch before tagging.** The whole
   point of integrating before shipping is to catch problems that
   only show up when fixes interact. Run `./scripts/preflight.sh`
   on the deploy branch (pytest, tsc, lint, vitest). Then do a
   manual sanity sweep: launch the app from this branch and exercise
   each fix's user-visible path. If any fix regresses something,
   drop it from the branch (git revert the merge commit) and keep
   shipping the rest. Don't tag until the deploy branch is green
   and manually verified — main never had the problem and the
   deploy branch is the only place we can catch it.
6. **Tag the release.** `git tag v<X.Y.Z>` on the deploy branch's
   tip and push the tag. The release commit's body becomes the
   GitHub release notes — the workflow extracts it via
   `git log -1 --format=%b`, so write user-facing notes there
   (not the engineer-facing "what files changed" kind).
7. **Run the release pipeline.** GitHub Actions builds the three
   platform installers and creates a **draft** release on GitHub
   with the notes auto-populated from the tag commit body.
8. **Sign the installers.** Run `scripts/sign-release.sh v<X.Y.Z>`.
   This downloads each installer from the draft release, signs each
   one with the maintainer's local minisign key (you'll be prompted
   for the passphrase per artifact), and uploads `.minisig` sidecars
   back to the same draft. The auto-updater on existing installs
   verifies these before launching anything; without them, the
   "Install now" click on every installed copy returns an error
   instead of running the unsigned binary. See
   `docs/release-signing.md` for first-time key setup, key rotation,
   and recovery scenarios.
9. **Publish the draft.** Open the Releases page, confirm both
   installers AND `.minisig` sidecars are listed under Assets, skim
   the auto-populated notes, click Publish. Drafts are invisible to
   the auto-updater (`/releases/latest` excludes them), so a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [J-M-PUNK/tideway](https://github.com/J-M-PUNK/tideway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
