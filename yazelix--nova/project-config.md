---
trigger: always_on
description: This file is self-contained. Canonical protocol text was rendered into it;
---

# Agent Guidelines

This file is self-contained. Canonical protocol text was rendered into it;
the source repository is needed only to update or verify the import.
Do not edit this generated file directly. Edit `.agent-protocols.local.md`
or `.agent-protocols.exceptions.json`, then render from the pinned source.

## Protocol import record

- Source: `https://github.com/Yazelix/starcompass`
- Source commit: `95c29fa76a971726b65e1d1dc06c518d525c46a2`
- Profiles: `orchestrator`, `release`, `fork`
- Manifest: `.agent-protocols.json` (schema 1)

| Protocol | Version | SHA-256 |
| --- | ---: | --- |
| `AP-SCOPE-001` | 1 | `b3f7e012df0708d4baf8957e3c315878a9eb8cd7fddf637dfde1506609d08444` |
| `AP-CONTRACT-001` | 2 | `f8a3b323f95816ee6d56d136c2ad19c691860155a4e58f57a62a4fb4dfc78cbb` |
| `AP-REFERENCE-001` | 2 | `ecb28af3796a9964dd98c037463a33b7444d9c39a0365783fb0e9ae9fc007b9c` |
| `AP-MINIMAL-001` | 2 | `3f4ebb1ab87f50ed5c9f041fb60d94cc7ded264ab04e56413fd1da343baa38f3` |
| `AP-DEPENDENCY-001` | 1 | `a389ff9054708574c52ec5e5dd7fc3e2d13b125d2218c70062f50a86761981ca` |
| `AP-OWNERSHIP-001` | 1 | `bdc09117f79b0d8dbe78e2dd8673a2463398aa31880fe27209fd6cc47f58bbf7` |
| `AP-TEST-001` | 1 | `363da7c542521be22233a4cc3373c0d3c3c5a9a0cf37f633cd5029545f4a3bee` |
| `AP-PROOF-001` | 1 | `1af235a56e9711d55362d869fa4057f1658d0aa7fe766030be0897ee5fd7c02b` |
| `AP-PLAN-001` | 6 | `04b96c47978d49d2e6584f89626bca42b6cb39df537fa3e5abd6da0bdc2edfda` |
| `AP-CI-001` | 1 | `78f1662259cd83f33d22ff4ddd0859ab0d4f704ba4f38756eef40a8b9b787bec` |
| `AP-ORCHESTRATOR-001` | 1 | `6228280f59c46dae79e45986d3848f071fd57699c8ce548ec2f7e503689e09c8` |
| `AP-FRONTIER-001` | 1 | `60e397fa4862afe902792501ffc6d2a82c94479877196b4278e7283a20111fe0` |
| `AP-PORTABILITY-001` | 1 | `d2800376013bbe1ade3449f835daf8780e60d4c08a79a7f733d4a651c4d6d887` |
| `AP-EXCEPTION-001` | 1 | `f66749229dbbc005e1c3103bfed86cf95169d7b32466c72fd8442e841cadb268` |
| `AP-GIT-001` | 3 | `16d27b0df7ccc94880bb31020e822e32b37503f43c2cf7a69de333300cbfdacf` |
| `AP-DELIVERY-001` | 1 | `5e424f8f1adf9dc86864e3c0e209b47ed577e17baa74e3ac2219e2209a4ff80b` |
| `AP-PROMOTION-001` | 1 | `d75780b1ee1dd658f32c80ff5edda5bff342212569029d0eb3bdc382f471d66e` |
| `AP-FORK-001` | 1 | `c8137dfbdc699c78cf7d8cd8a7dff7280b4013a84ed65c9234e7fb6511ca4563` |
| `AP-FORK-002` | 1 | `d896c5c45cf1615da683fa7503365d8d58286d4b7198ffc49ef9b9a0631b9ef1` |

### Local exceptions

No local exceptions.

## Canonical protocols

### AP-SCOPE-001 — User-owned scope

The user decides product and project scope. An agent may inspect, explain, test,
or make the smallest implementation needed for the chosen goal, but it must not
silently create a feature, compatibility promise, public surface, migration,
repository, or planning item outside that direction.

Required practice:

- Separate safe implementation details from choices that change product scope.
- State consequential assumptions; stop when a missing choice would materially
  change the result.
- Treat a terminal instruction such as “finish” as persistence, not broader
  authority.
- Keep useful out-of-scope observations as findings unless the user has chosen
  a durable planning destination for them.

### AP-CONTRACT-001 — Contract-driven changes

State the irreducible externally observable behavior before choosing the code
shape. Give durable contracts stable identifiers when later code, tests, or
repositories need to cite them.

Required practice:

- Among contracts that fit the user request and available evidence, choose the one
  with the fewest unsupported guarantees or restrictions. Leave reasonable
  future behavior unspecified unless the request or evidence requires a
  commitment.
- Name the consumer, trigger, observable result, and important failure behavior.
- Identify the current sources of truth and decide which one owner survives.
- Choose the cheapest check that can falsify the contract.
- Implement the smallest vertical slice that satisfies it.
- Update the contract first when an intentional behavior change is chosen.

Do not turn implementation details into contracts unless another component must
rely on them.

### AP-REFERENCE-001 — Evidence before code shape

Review the relevant sources before deciding architecture or implementation
shape. Memory, summaries, and reputation are discovery aids, not sufficient
evidence for a consequential decision.

Required practice:

- Read the affected local code, contracts, tests, and repository instructions.
- Inspect designated external references at the subsystem named by local rules.
- Record the concrete mechanism adopted, rejected, or left unresolved.
- Distinguish direct source evidence from inference.
- Revisit the evidence when the proposed shape changes materially.
- Apply source-license wording to the actors, uses, and conditions it actually
  names. Do not infer that an independent user or project acts on behalf of,
  for the benefit of, or under the direction of an agent or tool provider
  merely because the user selected that provider's service. Examples
  introduced by words such as “including” remain scoped by the condition they
  illustrate.
- Distinguish inspecting public source for ideas from copying, adapting,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Yazelix/nova](https://github.com/Yazelix/nova) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
