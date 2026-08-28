---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

CraigVault is a password-protected text editor that lives entirely in one HTML file — **and so does the document**. Saving writes a new HTML file carrying the editor plus the user's notes as AES-256-GCM ciphertext; double-clicking that file boots straight into a password prompt. Zero network requests. `index.html` is the blank template; every saved vault is a standalone copy of it with a payload.

## Commands

There is **no build step, no test runner, no linter, and no `package.json`**. Don't look for them.

```bash
xdg-open index.html            # run it — file:// is a secure context, so crypto.subtle works
python3 -m http.server 8000    # serve over HTTP when you need a real origin
```

In-place saving needs the File System Access API (Chromium only). Firefox and Safari exercise the download fallback, which is a separate code path — see gotcha 8. **Embedded browser views (VS Code's Simple Browser, and anything else that frames the page cross-origin) also fall back**, because Chromium refuses the picker there — see gotcha 12. Test in a real browser window, not the editor's preview pane.

## Architecture

Everything is `index.html`, 900 lines, three layers:

- **Styles** (`:7-118`) — CSS custom properties on `:root` are the whole design system (brass `#c9a227` accent on dark panels). No framework.
- **Markup** (`:120-253`) — header toolbar, editor `<textarea>`, an absolutely-positioned `.lockscreen` overlay, and three `<dialog method="dialog">` elements (`pwDialog` for setting a password, `openDialog` for entering one, `confirmDialog` for destructive confirmations). A `#toast` div carries transient status messages.
- **Payload** (`:253`) — `<!--CRAIGVAULT:BEGIN--><script id="vault" type="text/plain">…</script><!--CRAIGVAULT:END-->`, the base64 document. Empty in the template.
- **Script** (`:255-898`) — banner-commented sections: `PRISTINE` capture, crypto core (scrypt + AES-GCM), vault container, app state, transient messages, password dialogs, file ops, lock/unlock, idle auto-lock, wiring, boot.

**State model.** Three states, driven by `password` (`:488`) and `locked`. Note the script at `:255` is a *classic* script, not a module, so these top-level `let`s are global lexical bindings — the DevTools console resolves them by name, and `function` declarations land on `window`. Anything left in a variable is reachable, which is why locking clears rather than merely hides:

```
no document  →  unlocked                    →  locked
(no password)   (password set,                 (plaintext wiped from the DOM,
                 plaintext in the textarea)     password back to null,
                                                ciphertext held in lockedBlob)
      ↑                                              ↑
 blank template                            boot() lands here directly when
 (empty payload)                           the file carries a payload
```

`boot()` (`:878`) is the entry point: a payload means come up `locked` with `lockedBlob` set and no `password`. That is the *same* state `doLock` produces — locked always means no key in memory, however it was reached — so there is exactly one locked state to reason about, not two. `doUnlock` adopts the password on success, which is what makes it resolvable in both cases.

**Crypto contract** (`:262-426`). AES-256-GCM for the cipher, **scrypt** (RFC 7914) for key derivation. `encryptText` / `decryptBytes` / `deriveKey` are the only functions that touch `crypto.subtle`. Salt and IV are freshly generated on every save. `decryptBytes` throwing is the *only* wrong-password signal — GCM authentication doubles as the password check, so there is no separate verifier to maintain.

scrypt is PBKDF2-HMAC-SHA256 bookends around a memory-hard core, and both bookends run at **one** iteration, so `crypto.subtle` still does every hash. The only hand-written cryptographic code is `salsa20_8` (`:299`), `blockMix` (`:325`) and `roMix` (`:337`) — pure arithmetic, checked against the RFC 7914 vectors. **Do not reimplement a hash function here.**

Measured in Chrome: N=2^14 → 124ms, 2^15 → 245ms, 2^16 → 485ms; the PBKDF2 it replaced was 74ms. Shipping `LOGN = 15` (`:286`) — 32 MB per guess. The point is not wall-clock but memory: PBKDF2 let a GPU run thousands of guesses in parallel almost free, scrypt makes each one cost 32 MB.

## Hard constraints

- **Never add a dependency, bundler, framework, or `src/` split.** One self-contained file with nothing to trust is the product's security argument, not a stylistic preference. Push back on proposals that break it.
- **Never persist the password or plaintext.** No `localStorage`, `sessionStorage`, IndexedDB, cookies, or network calls. The password lives in a variable that dies with the tab.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [craig-stevenson/craigvault](https://github.com/craig-stevenson/craigvault) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
