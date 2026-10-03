---
trigger: always_on
description: This file applies to the whole Linnet repository. Its purpose is to keep the
---

# Linnet Engineering Discipline

This file applies to the whole Linnet repository. Its purpose is to keep the
product small, direct, testable, and fast to change. User instructions override
it. A closer path-specific `AGENTS.md` may add constraints but must not weaken
these rules.

## 1. Failure Pattern This Document Prevents

Previous work became slow and over-engineered because uncertainty was converted
into code: another guard, wrapper, fallback, parser, compatibility branch,
state machine, test harness, or release gate. Local desktop inputs were treated
like hostile network traffic, tests were used as evidence of product quality,
and builds were repeatedly started before the code change was complete.

These are process failures, not acceptable safety margins. Linnet must prefer a
small correct product path over speculative resilience.

## 2. Product First

Before editing production code, write one compact milestone containing:

- the user-visible behavior to deliver;
- the current authoritative owner and every directly affected caller/consumer;
- the earliest proven cause of the defect;
- the old path, branch, helper, or layer that will be deleted or de-authorized;
- the allowed files and the focused acceptance command;
- whether loaded macOS behavior, local iCloud, or VM lifecycle UAT is required.

Do not start implementation while the cause is only a screenshot-level
hypothesis. Trace input event -> Linnet owner -> Rime/AppKit boundary -> visible
result. Fix the earliest wrong owner, not a downstream symptom.

Normal typing is the primary acceptance boundary. Candidate selection, input
continuity, input-menu visibility, application connections, and no-logout Core
updates take priority over internal abstractions and defensive checks.

## 3. One Owner, One Path

Each fact or state transition has one authoritative owner. A change must not add
a second producer, fallback, alias, UI inference, compatibility reader, or
pass-through adapter for an existing fact.

For every production change, record before/after counts for:

- authoritative paths;
- pass-through layers;
- fallback and compatibility branches;
- duplicated defaults and raw inference sites.

The default accepted result is flat or lower. A helper or new file is allowed
only when it owns a new invariant/external boundary or removes an existing
branch, interpretation site, or call-chain hop in the same change.

Do not add a wrapper around behavior that already has an owner. Do not retain an
old implementation “just in case.” If a shipped external contract truly needs
temporary compatibility, name the contract, owner, expiry, and removal test.

## 4. Complexity Control

For an ordinary bug or behavior correction:

- freeze the design before editing;
- use at most two corrective implementation loops;
- finish the code edit as one batch before building.

File count, line count, and elapsed time are diagnostic signals, not acceptance
limits. A large diff should prompt one scope review, but must not be split,
compressed, or rewritten merely to satisfy a number. Never trade readable,
complete behavior for fewer files or lines. Accept or reject a design by whether
it has one owner, removes replaced paths, avoids duplicated decisions, and
preserves the complete product contract.

When a change grows unexpectedly, stop adding code and re-read the complete
owner/call chain. Continue with the current design when the size is inherent to
the requested behavior; redesign only when the growth comes from duplicated
owners, speculative branches, adapters, or unrelated scope.

Do not create custom parsers, package formats, IPC protocols, daemons, caches,
transaction systems, layout frameworks, or recovery state machines when a
standard Foundation/AppKit/Rime facility satisfies the actual product contract.
Existing public formats may be retained, but must not grow without a new
user-facing requirement.

Prefer deletion and direct calls. File splitting is not a refactor when total
logic, branches, and ownership remain unchanged or grow.

## 5. Proportionate Safety

Every retained safety check must state:

- the concrete trust, time, or mutation boundary it protects;
- the actual input and credible failure mode;
- the action it blocks;
- why an existing check does not already protect that boundary.

User-owned local settings and personal dictionaries are not public hostile
network services. Do not add anti-DoS parsers, arbitrary quotas, repeated full
hashing, recursive normalization, or multi-layer validation without measured
evidence and a real boundary.

Downloaded release metadata, packages, filesystem ownership transitions,
cross-process mutations, code signing, and publication are distinct external or
mutation boundaries and may be validated once by their canonical owner. Later
consumers must not repeat or reinterpret the same decision.

Restarting, clearing caches, or deleting state is not diagnosis. Preserve the
earliest evidence first.

## 6. UI And Performance

Use AppKit's normal lifecycle and lightweight frame/layout facilities before
introducing delegate proxies, runtime forwarding, nested view bridges, or custom
rendering infrastructure. Keep Rime as the candidate/order/selection owner; UI

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ares-X/Linnet](https://github.com/Ares-X/Linnet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
