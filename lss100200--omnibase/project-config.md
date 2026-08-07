---
trigger: always_on
description: This file is the repository entry point for coding agents. Its purpose is to
---

# OmniBase AI Maintainer Contract

This file is the repository entry point for coding agents. Its purpose is to
make OmniBase repairable from the public source tree without relying on a
private conversation, hidden workspace state, or an original maintainer's
memory.

## Read order

Before changing architecture, authentication, tenancy, migrations, P34, SDKs,
or recovery tooling, read these sources in order:

1. `AGENTS.md`
2. `docs/maintainers/maintenance-map.json`
3. `docs/maintainers/security-invariants.md`
4. `docs/maintainers/ai-maintainer-map.md`
5. The module source, migrations, contracts, and tests named by the map
6. `docs/handover-report.md` for current phase status and verified evidence

If prose and executable behavior disagree, do not guess. Treat source,
database constraints, migrations, contract snapshots, and passing tests as the
runtime evidence; then correct the stale documentation in the same change.

## Non-negotiable boundaries

- Browser and user API traffic enters through `backend/src/omnibase/main.py`
  under `/api/v1`. The root `/health` route is a probe-only compatibility path.
- The Capability Gateway is a separate non-browser ASGI application created by
  `backend/src/omnibase/capability_gateway/app.py:create_gateway_app`. Its
  defaults reject all workloads until trusted attestation and verification
  components are injected. Do not silently mount it into the browser app.
- JWT claims are necessary but not sufficient. Protected browser requests must
  revalidate the live tenant, live user, current role, and tenant schema through
  `backend/src/omnibase/tenants/dependencies.py:get_current_principal`.
- External callers use logical resource, table, column, operation, and grant
  identifiers. Physical PostgreSQL schema/table/column locators must remain
  server-owned and must not appear in public DTOs, SDKs, logs, or errors.
- Capability authorization is tenant/workspace/runtime/action/resource/version
  bound, time limited, budget limited, revocable, and fail-closed. Never replace
  verification with possession of a raw identifier or a browser cookie.
- High-risk operations, approval consumption, idempotency, auditing, and data
  mutation are one security system. Do not update one record in isolation when
  the current service performs an atomic lifecycle transition.
- Audit records are append-only. Migration `0006` installs database enforcement;
  application code must not treat ORM discipline as the only protection.
- Migration scope is a closed set: `global` or `tenant`. Unknown or missing
  scope must fail closed. Production recovery is forward-fix or restore into a
  new `omnibase_restore_*` database, never destructive in-place guessing.
- P34.4 Workspace governance, lifecycle metadata, lease/fencing, trusted-node
  control records, and the fake/local collaboration harness are explicitly
  unlocked. P34.5A0-A4/B/C/D are engineering-sealed: the independent Hyper-V
  Linux Runner passed its 11/11 isolation Gate, the independent PrivateNetwork
  Broker daemon passed two 26/26 namespace/default-deny/identity/budget/replay
  rounds, the Headscale adapter passed a real control-plane Gate with an mTLS
  Node-Daemon test double, and the split-process mTLS Gateway passed the guarded
  disposable schema/rows/RAG/citation read Gate. Production defaults still
  reject until the documented trusted wiring is explicitly assembled. This
  does not authorize a normal Docker/WSL host to run hostile code, expose a
  member Overlay endpoint to a Sandbox, connect a Sandbox or Runner directly to
  PostgreSQL/Redis/MinIO, or treat the disposable Gates as proof of production
  Core-to-Runner/Broker activation, non-disposable tenant/RAG, real member data
  plane, DERP, node-compromise, capacity, or SLA readiness. Workspace
  private-write/RAG promotion and Agent Runtime/orchestration remain frozen
  until the roadmap and handover explicitly unlock them.
- P34.4 membership mutations serialize on the tenant-bound Workspace aggregate,
  then re-lock the actor and target membership before evaluating the active-owner
  invariant. Template registration revalidates the live tenant administrator in
  the caller-owned transaction and uses the PostgreSQL natural key for exact
  concurrent replay. Do not replace either rule with a pre-transaction role
  snapshot, an unlocked owner count, or catch-and-ignore `IntegrityError`.
- Run and Network leases are independently fenced. A Run lease is bound to the
  current Node fencing token and a Network lease is a logical authorization
  allocated from `network_lease_cursors`; P34.4 never activates a provider while
  signing that logical Network lease. Every use revalidates the current live
  attestation. Terminal Runs cannot return to a running state or retain runtime
  or workload identity metadata.

## Safe change workflow

1. Locate the target module in `docs/maintainers/maintenance-map.json`.
2. Read every listed invariant and dependency before editing.
3. Preserve tenant predicates, logical/physical identifier separation,
   transaction boundaries, lock order, idempotency, audit writes, and
   fail-closed behavior.
4. Make the smallest change that fixes the proven issue. Do not hide type

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lss100200/omnibase](https://github.com/lss100200/omnibase) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-03 -->
