---
trigger: always_on
description: SPIRE is the reference implementation of the SPIFFE APIs. It issues SPIFFE
---

# Copilot instructions for SPIRE

SPIRE is the reference implementation of the SPIFFE APIs. It issues SPIFFE
identities (SVIDs) to workloads in the SVID formats defined by the SPIFFE
specifications. The codebase is Go and is organized around two long-running
processes, the SPIRE Server and the SPIRE Agent, plus a plugin system.

When reviewing or generating code, weigh the concerns covered below. All of
them matter, and more than one often applies to the same change: compatibility
guarantees, SPIFFE spec conformance, security and usability, and project
conventions.

## Review checklist

These questions come from the SPIRE maintainer review guidelines in
`MAINTAINERS.md` and are advisory prompts to surface issues, not a rigid gate:

- Is the use case the change addresses clearly understood?
- Does the change break any current user's expectations of behavior (a
  regression)?
- Can the change be misconfigured, and if so what is the impact?
- Does the change adhere to the SPIRE compatibility guarantee (see the
  Compatibility guarantees section below)?
- What are the failure modes? Can SPIRE keep running?
- If something goes wrong, will it be clear to the operator what happened and
  how to fix it?
- If the change introduces additional configurables, could some or all of them
  be replaced with a programmatic decision?

Changes to particularly sensitive areas, such as the agent's cache manager or
the server's CA, warrant extra scrutiny.

## Compatibility guarantees

SPIRE makes strong compatibility promises documented in `doc/upgrading.md`.
Treat a violation of these as a blocking issue.

- **Server-to-server version skew.** Servers in the same cluster must operate
  within +/- 1 minor version of each other. Do not introduce a change that
  breaks a server one minor version behind or ahead (for example a new field or
  RPC that an adjacent server cannot produce or understand).
- **Agent-to-server version skew.** Agents may be up to one minor version older
  than the oldest server and must never be newer than the oldest server. Do not
  add behavior that requires an agent and server to be the same version, or that
  breaks an agent one minor version behind.
- **Upgrade and downgrade paths.** Only single-minor-version jumps are
  supported, servers upgrade before agents, and SPIRE supports zero-downtime
  rolling upgrades when more than one server is present. Do not assume state
  written by a version more than one minor prior.
- **Built-in plugin config and behavior compatibility.** A backwards-incompatible
  change to a built-in plugin (config semantics, selectors produced, etc.) must
  log a deprecation warning and keep backwards compatibility for one minor
  version. Do not rename, retype, or remove a config key without a deprecation
  cycle.
- **Plugin interface compatibility.** A breaking change to a plugin interface
  must keep existing plugins working for one minor version cycle, with warnings.
- **Deprecation log markers.** Deprecation warnings must include the structured
  field `alert=true`. Configuration deprecations add `alert_type=deprecated_config`
  and deprecated plugin services add `alert_type=deprecated_service`.
- **Datastore/SQL changes.** Datastore schema changes must ship in at least one
  full minor release cycle before any code change depends on them.
- **Experimental features.** Features gated behind the `experimental` config
  section are exempt from the guarantees above. If a change touches a feature
  that was experimental, confirm the experimental marker is still accurate.

## SPIFFE specification conformance

SPIRE implements the SPIFFE specifications. The canonical, authoritative set of
specs lives at https://github.com/spiffe/spiffe/tree/main/standards. The set of
specifications and SVID profiles evolves over time, and new profiles are added,
so do not assume the specs are limited to the ones you already know from
training.

When a change touches spec-defined behavior (an SVID format, trust domain
parsing, the Workload API request/response shape, federation bundle format,
etc.), treat the specs in that repository as authoritative and verify
conformance against them rather than relying on prior knowledge, which may be
stale or incomplete. If you cannot consult the spec, raise spec-conformance
concerns as questions rather than asserting a defect. Likewise, flag
reinterpretations of ambiguous spec areas as questions for maintainers rather
than as defects.

## Security and usability

SPIRE is security-critical infrastructure. Watch for changes that could weaken
identity issuance, trust domain or bundle handling, SVID validation, key
management, or authentication and authorization between components. Consider
whether a change could be misconfigured, and what the impact of misconfiguration
would be. Vulnerabilities are reported privately to security@spiffe.io, not via
public issues or PRs.

SPIRE solves a complicated problem, so features, configurables, log and error
messages, documentation, and naming must stay accessible to people who are not
deeply familiar with SPIFFE or authentication systems. The maintainer guidelines
in `MAINTAINERS.md` set these expectations:

- **Secure by default, then "it just works".** Decisions should favor secure

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [spiffe/spire](https://github.com/spiffe/spire) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
