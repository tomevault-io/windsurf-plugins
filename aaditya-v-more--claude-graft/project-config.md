---
trigger: always_on
description: Comments say **why**, never what. Delete a comment that restates the line under it.
---

# Working on Claude Graft

## Style, first

Comments say **why**, never what. Delete a comment that restates the line under it.

    // WRONG — increments the counter
    failures += 1

    // RIGHT — a refused call retried on the next tick becomes 120 attempts an
    // hour, which is how a client earns a rate limit rather than avoids one.
    failures += 1

Prose in the README, not bullet lists. Tests read as sentences — `check(…, "a
profile with no stored login has no token")`, not `"testTokenNil"`.

Nothing that leaves this machine may name a tool, a model or a pipeline. That
includes commit messages, the README and anything pushed. Write what a person
would have written by hand.

## Commands

    ./build.sh                 app + launcher into build.noindex/, this arch only
    GRAFT_UNIVERSAL=1 ./build.sh   both arches, joined with lipo
    ./test.sh                  346 checks, all in a throwaway directory
    ./release.sh [--install]   tests, builds universal, draws the icon, signs, packages

## Invariants

**Never touch an app Graft did not create.** Bundles are identified by the
`graft.json` they carry, never by name. `Claude` and `Claude Graft` are reserved
names. This rule exists because an earlier version matched on name and deleted
`/Applications/Claude.app`.

**Never destroy what a profile owns.** Anything a link replaces is moved to a
hidden `.<name>.graft-own` sibling first, and restored when the shortcut goes
back to its own chats. A profile must never be grafted from itself — guarded in
`graft`, `relink` and `install`, because self-grafting stashes away every file
the profile has and leaves links pointing at their own names.

**A stash already sitting there is not proof of a duplicate.** Claude writes
`config.json`, and recreates chat directories, by renaming a temporary over
them, and a rename leaves a real file where the symlink was. The profile then
goes back to writing its own copy while the stash still holds the pre-graft
state. `stash` used to read that as drift and delete it, which threw away every
chat written since the graft; it now folds the two together and keeps the copy
the profile is actually using. `unstash` used to bail whenever something already
sat at the link, which is what left the stash there to arm the next graft.

**An unreadable `config.json` is not an empty one.** A profile that has never
been signed in has no file at all and can share the whole store safely. A file
that will not parse is one caught mid-rename, and treating that as "no account,
so the same account as the source" linked an entire store away. `readableConfigJSON`
tells the two apart; nothing may write that file, or decide where a profile's
chats go, without asking it.

**Credentials are borrowed, never taken.** Only the access token is decrypted,
never the refresh token: Anthropic rotates those and using one signs Claude
Desktop out. Nothing is written back to Claude's config, cookies or keychain.
The token goes to `api.anthropic.com` and nowhere else.

**The keychain is asked once, and only when asking is the way through.** Every
read tries silently first, so a build already on the item's ACL never sees a
dialog no matter who asked. When that read comes back shut the app asks rather
than falling back without saying why — once for the life of the process, never
again after a decline, and not while it is coming up hidden into the menu bar,
since a dialog thrown over a login is answered by nobody. The rule is
`ClaudeCredentials.mayRaiseDialog`, kept pure so the suite can drive it;
`UsageMonitor.mayPromptUnasked` carries the login case.

`SecKeychainSetUserInteractionAllowed` is what actually makes a read silent. The
`LAContext` the compiler suggests in its place governs items backed by an access
control, not an ACL of trusted applications, and was measured sailing straight
past the suppression and putting a dialog on screen from a background poll —
which is what the old "background polls never prompt" rule believed it had.

**Only a person starts a session.** `SessionStarter.start` runs because a
button was pressed and for no other reason: it defaults to non-interactive, so
a call from anywhere unaudited fails quietly instead of raising a keychain
prompt, and it claims the profile for the length of the call so the window and
the dropdown cannot both start the same account. The suite reads
`Sources/App/*.swift` and fails if a fourth caller appears or if `startSession`
turns up on a line with `onAppear`, `onReceive` or a timer — a view refreshes
every thirty seconds, and a session wired into one would open a five-hour
window on every account, over and over.

**⌘Q does not quit.** `applicationShouldTerminate` cancels an unasked-for
terminate and closes the window instead, because the menu bar item is the part
that does the reporting and closing a window is not asking for it to stop.
`AppDelegate.quit()`, wired only to Quit in the dropdown, is what really ends
it, along with a logout — refusing that stalls the shutdown on a dialog — and
along with an update. Sparkle starts the new version by asking this one to
terminate; refusing that left 1.0.0 running with 1.0.1 already staged, put the
window away on the way past, and wrote `mainWindowOpen` false, so every launch

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aaditya-v-more/claude-graft](https://github.com/aaditya-v-more/claude-graft) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
