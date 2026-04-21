---
trigger: always_on
description: This file provides guidance for contributors and coding agents in this repository.
---

# AGENTS.md

This file provides guidance for contributors and coding agents in this repository.

## What is ReMemory

ReMemory encrypts files with [age](https://github.com/FiloSottile/age), splits the decryption key among trusted friends using Shamir's Secret Sharing (via HashiCorp Vault's implementation), and gives each friend a self-contained offline recovery tool (`recover.html`) that works in any browser without servers or internet.

## Ownership Mindset

**You own the outcome, not just the task.** This isn't typical software where bugs get patched next sprint. A recovery bundle might sit in a drawer for ten years, then be opened by someone who just lost a loved one. You won't be there to fix it. The code you write today must work perfectly for someone you'll never meet, in circumstances you can't predict.

This changes how you work:

- **Search before creating.** Before adding a constant, helper, or type, grep for where it might already exist. Data should have one source of truth. If you need a list of values, find where the authoritative list lives and derive from it — don't create a second copy that will drift.
- **Trace all connections.** When removing or changing something, find every reference: code, styles, translations, types, tests, comments, build config. Use grep. Removing a feature means removing the HTML elements, the CSS classes, the translation keys, the JS functions, the tests — all of it. If you're surprised by leftover references after making a change, you didn't trace thoroughly enough.
- **Delete, don't hide.** If something shouldn't exist, remove it completely. Don't comment it out, don't hide it with CSS, don't wrap it in a conditional. Dead code is confusing code.
- **Verify before claiming done.** After any change, grep for related terms, rebuild, run tests. You should never be surprised by leftover references — you should have found them first.
- **Understand the system before changing it.** Read the existing implementation. Understand why it works the way it does. Match its patterns and standards.

The goal is simple: when someone opens `recover.html` years from now, everything should just work. That requires code written by someone who cared enough to get every detail right.

## Development Principles

- **Care and attention to detail.** This software protects important information for real people. Mistakes can mean lost secrets, failed recoveries, or leaked data. Being thorough means checking your own work, matching the standards already in the codebase, and not leaving loose ends for others to find.
- **Empathy.** The people recovering secrets may be non-technical, stressed, or grieving. Every message, instruction, and UI choice should be clear, patient, and helpful. Lend a hand, don't assume expertise.
- **Stand the test of time.** Recovery bundles may sit untouched for years or decades before they're needed. Avoid dependencies on external services, ephemeral formats, or assumptions about the future. The bundles must work even if this project disappears.
- **Universality.** The recovery experience must work across platforms, browsers, and languages.
- **Considered tone.** When writing user-facing copy, every word should feel placed, not emitted. Stay honest about what this tool is and isn't. Don't oversell or make grand claims. See the **Voice & Copy** section below for detailed guidance.
- **Shared logic across CLI and browser.** Cryptographic operations and core logic live in `internal/core/` and are reused by both the CLI and browser paths. Don't duplicate — centralize. Before creating a new constant or helper, grep to see if it already exists elsewhere. Data should have one source of truth.
- **Tests verify safety.** Write a failing test first, then make it pass. This applies everywhere — Go unit tests, integration tests, and Playwright browser tests alike. If you can't demonstrate the test failing without your change, you can't be sure it's actually testing anything. Any change that touches `recover.html` or `maker.html` needs a corresponding Playwright test. Run the tests yourself before calling something done — don't leave verification to others.
- **Keep docs current.** When changing behavior, update the relevant docs, README, and this AGENTS.md file in the same change.
- **No network in recovery.** `recover.html` must not make network requests. Avoid adding dependencies that could pull remote resources (fonts, CDNs, analytics, etc.).
- **Stable formats.** The share format, bundle layout, and recovery steps are part of the protocol. Changing them requires migration thinking, updated test fixtures, and clear rationale.

## Feel

Remember who is reading this. Someone creating bundles may be confronting their own mortality, planning for the worst, or already dealing with loss. Someone recovering files may have just lost a person they love. They may be scared, overwhelmed, or grieving. They are not "users" in the normal sense. They are people in a hard moment, trying to do something important.

The design should make them feel safe, guided, not rushed, not judged. Soft, warm, paper-like, non-corporate. Low contrast by design: easy on stressed eyes.

### Voice & Copy


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eljojo/rememory](https://github.com/eljojo/rememory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
