---
trigger: always_on
description: An **authorized-testing** Android app pentest pipeline built as a small set of **agentic skills**
---

# CLAUDE.md — Tribunal (Android app pentest pipeline)

An **authorized-testing** Android app pentest pipeline built as a small set of **agentic skills**
(not a heavy multi-stage harness), designed to be cloned and run against any APK with **zero
configuration**. Skills pass structured JSON through `targets/<apk-sha256>/` and keep raw
decompiled output (`apktool/`, `jadx/`) on disk for pivoting. The name comes from the last stage:
nothing reaches `report/` without surviving an adversarial prosecution/defense/judge trial first.

## The workflow
1. **Drop an APK into `apk/`.** That's the only setup step.
2. Trigger the pipeline (say "test this APK" / "run the pipeline" / invoke **pentest-runbook**).
   It hashes the file, skips straight to whichever stage `targets/<hash>/` hasn't finished yet
   (idempotent, resumable), and otherwise runs **stage 1 → 2 → 3 → 4 back-to-back with no manual
   gate in between.**
3. Every finding that survives dynamic/API confirmation goes to courtroom-verdict (stage 4) —
   only ones a judge subagent rules `guilty` after an adversarial prosecution/defense trial become
   a written report at `report/<package>/<finding-id>-<slug>.md`. Not just candidates: findings
   whose verification plan has already been executed, evidence captured under `evidence/<id>/`,
   and exploitability argued and independently checked.
4. Clone-and-go: a fresh clone of this repo, handed just an APK in `apk/`, reproduces the same run.

## Non-negotiable rules
- **`apk/` is the scope boundary — nothing else is.** A skill may only ever decompile, install, or
  test an APK file that physically resides inside `apk/`. Placing a file there **is** the
  operator's assertion that they own the app or are authorized to test it. There is no
  scope-list to maintain and nothing to add anyone to. Fail closed on the *path*, not a package
  registry: refuse any APK/path that resolves outside `apk/` (no `..`-escapes, no fetching a
  target from a URL or another package's data). We removed the old `scope/scope.json`
  allow-list model on purpose — don't reintroduce a package registry as a substitute gate.
- **Your emulator, your device.** Dynamic stages (2–3) run against an emulator/device the
  operator controls. This is a *where*-you-test rule, separate from the *what's-in-scope* rule
  above — it's about not accidentally driving Frida/Burp against a device you don't own.
- **Methodology + standard tooling only.** Writing a **custom Frida SSL-pinning bypass for an app
  you installed on your own emulator is in scope and expected.** Do NOT build tools whose purpose
  is attacking apps outside the `apk/`-is-scope model above. No weaponized companion apps.
- **Fully autonomous, end to end.** Stage 1 (static) never touched a network to begin with. Stages
  2–3 (dynamic + API) are equally autonomous now — no per-finding confirmation step — because the
  authorization question was already answered in step 1 above (the file is in `apk/`).
- **Preflight, every dynamic stage, first.** Stage 1 needs no device. Before stage 2 (and again
  before trusting stage 3's live traffic), run `scripts/preflight_check.py` — confirms adb sees a
  device, it's an emulator, frida/objection are on PATH, and (optionally) frida-server is already
  running. NOT READY (exit 3) means stop and fix the printed gap, not push ahead and improvise
  around a half-working toolchain.
- **No deterministic sink-grep seeding the review.** `inventory.py` only decompiles + inventories
  (file counts, framework fingerprint, obfuscation detection). The actual review — building the
  threat model and finding vulnerabilities — is done by agents reading the decompiled tree
  themselves, not by a regex hit list. See Architecture, stage 1.

## Architecture
Three focused skills + an orchestrator, composing via `targets/<hash>/`:
1. **decompile-threat-review** — apktool+jadx → `targets/<hash>/` (deterministic mechanics only:
   decompile + inventory, no sink-grep). Then a **main agent fans out subagents** — by default,
   not opt-in — to build the threat model and hunt vulnerabilities: one subagent reads the
   manifest + source to build the structural threat model, independent lens subagents each review
   a disjoint slice of `docs/vuln-catalog.md` (the 13-category rubric) in parallel, and skeptic
   subagents try to refute every candidate before it ships. Emits `threat_model.json` +
   `report.json`/`report.md` (each finding carries a dynamic verification plan). See
   `.claude/workflows/android-adversarial-review.js`.
2. **dynamic-verify** — preflight, then install/launch on the emulator, proxy→Burp + CA, get
   traffic flowing (generic unpinning first; if that fails, a **dedicated subagent** reads
   `threat_model.json.pinning` and the actual source at that location — not just the class name —
   and writes a tailored hook to `targets/<hash>/frida/bypass.js`), execute each verification
   plan, capture `evidence/<id>/`,
   update finding `status`.
3. **api-vuln-test** — enumerate endpoints (threat_model + Burp), test IDOR/BOLA/auth/BFLA/
   injection/business-logic by replaying modified requests; emit `api_findings.json`.
4. **courtroom-verdict** — every finding that reached `status: confirmed` (client-side or API)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [abisheikM1/Tribunal](https://github.com/abisheikM1/Tribunal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-31 -->
