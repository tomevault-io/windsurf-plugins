---
trigger: always_on
description: This file guides Claude Code (claude.ai/code) and human contributors working in
---

# CLAUDE.md

This file guides Claude Code (claude.ai/code) and human contributors working in
this repository. Keep it concise — the full design lives in
[docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Project Overview

**cdk-real-drift** (`cdkrd`) is a drift detect/revert CLI for AWS CDK /
CloudFormation. It detects when your **real** deployed AWS resources diverge from
your IaC intent — **including properties you never declared** in the template. That
undeclared-property dimension is the differentiator: `cdk drift`, CloudFormation
drift detection, `driftctl`, and `terraform plan` all compare only properties that
appear in the template, so an out-of-band change to a setting you never declared
(a bucket's `OwnershipControls`, a role's `PermissionsBoundary`, an extra inline
policy) is invisible to them. `cdkrd` reads the **full** live resource model via
Cloud Control API (with SDK overrides for CC-gap types) and reports — and can
revert — the divergence. No AWS Config required.

It is **reality vs intent**, not code vs template: it deliberately does NOT
reimplement `cdk diff`. The full design, rationale, and pipeline are in
[docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) (with [DESIGN.md](DESIGN.md) as the
terse companion and [docs/redesign-notes.md](docs/redesign-notes.md) for
pre-publication decisions).

### Core invariant: a clean deploy has ZERO potential drift

A freshly deployed, **un-mutated** stack must produce **zero** `[Potential Drift]`
on a first `check` (before `record`). Every value AWS assigns at creation — an
initial/default it materialized that the template never declared — is NOT a
divergence, so it must fold to `atDefault`, never surface. `[Potential Drift]`
is reserved for a **real divergence**: a value the USER changed, or one AWS
changed OUT OF BAND _after_ creation (e.g. enabling Application Signals adding IAM
permissions later). Anything else appearing there is a **fold gap (a bug)** — this
is exactly the false positive the `check`-output note + issue link (#581) ask users
to report. When a candidate default's status is uncertain, **RESOLVE the
uncertainty by investigation / live verification** (deploy a fresh minimal config,
observe what AWS assigns undeclared) — never leave a value surfaced as
"conservative"; that just ships the bug. A value the user never changed appearing on
a first `check` IS the bug, not an acceptable state — **do NOT rationalize leaving it
`undeclared` as "honest".** Shrinking a fixture's first-`check` noise from 51 lines to
"a few" is not a fix; **the target is zero.**

**Fold-strategy decision order** — the fold must PRESERVE out-of-band detection where
the value is meaningful, so escalate through these in order and stop at the first that
applies; reach for the next tier only because the prior one genuinely cannot express
the default:

1. **Equality-gated constant** (`KNOWN_DEFAULTS` top-level / `KNOWN_DEFAULT_PATHS`
   nested) — folds the exact default value and still surfaces any change away from it.
   The DEFAULT choice for any default that is a stable constant.
2. **Derived default** (`CONTEXT_DEFAULTS` = f(region), `ENGINE_DEFAULTS` = f(engine),
   or a value computed from a sibling / declared property — e.g. an EB Environment's
   `MaxSize` default is derivable from its `EnvironmentType` option: SingleInstance→1,
   LoadBalanced→4). When the default is not a single constant but a DETERMINISTIC
   FUNCTION of the declared inputs, COMPUTE it and equality-gate against the computed
   value — detection is still preserved. **Before labelling a default
   "context-dependent, can't fold", ask "can I DERIVE it from the declared inputs?" —
   usually you can.** Do not skip to tier 3 just because a constant does not fit.
3. **Value-independent** (`VALUE_INDEPENDENT_DEFAULT_TOPLEVEL_PATHS` & nested kin) —
   LAST resort, and it LOSES change-detection (folds any value). Use ONLY when the
   default genuinely cannot be pinned OR derived: AWS moves it over time (a platform
   AMI id, a versioned asset URL, a GA engine version), or it is a per-resource
   AWS-assigned identifier / cosmetic value. Acceptable only because the value is
   UNDECLARED — the user delegated it to AWS, so whatever AWS assigns is not user
   intent; a user who cares about it DECLARES it, which is then detected in the
   declared dimension. Never value-independent a value the user can meaningfully set
   and would want to catch drifting when a constant or a derivation could fold it.

## The 4-Verb Model

```bash
node dist/cli.js check  [<stack>...] [--all]   # detect drift (read-only)
node dist/cli.js record [<stack>...] [--all]   # snapshot undeclared state into the baseline file (KEEPS watching)
node dist/cli.js ignore [<stack>...] [--all]   # stop reporting chosen drift via .cdkrd/ignore.yaml (STOPS watching)
node dist/cli.js revert [<stack>...] [--all]   # write the desired value back to AWS (confirms)
```

- **`check` is the primary entry point.** Day to day the user runs only
  `cdkrd check` and acts from its interactive prompt — it establishes the first
  baseline (R141) and offers record / revert / ignore inline on what it finds
  (R121/R133). The standalone `record` / `ignore` / `revert` verbs are the SAME

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [go-to-k/cdk-real-drift](https://github.com/go-to-k/cdk-real-drift) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
