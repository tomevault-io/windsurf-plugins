---
trigger: always_on
description: Read [CONTRIBUTING.md](CONTRIBUTING.md), [feature status](docs/feature-status.md),
---

# Tin agent instructions

Read [CONTRIBUTING.md](CONTRIBUTING.md), [feature status](docs/feature-status.md),
and the relevant guide in [docs/README.md](docs/README.md) before changing a subsystem.
Current code and pinned workflow definitions are authoritative. This file records
cross-cutting implementation safeguards, not historical deployment logs.

## Scope and implementation

- Tin is a Python/FastAPI service with packaged HTML, CSS and JavaScript, Postgres
  product projections, Temporal orchestration, code.storage artifacts and E2B compute.
- Keep one explicitly registered workflow engine. Do not introduce a node-graph engine,
  frontend framework, provider fallback or plugin marketplace as incidental work.
- Preserve unrelated edits and use an isolated worktree when another agent is working.
  Do not deploy, change provider configuration, rewrite history or publish repositories
  without explicit authority. A merged PR is not a production deployment.
- Do not copy runtime dependencies from unrelated repositories. Keep the import-linter
  boundaries and the explicit registry; no wildcard workflow registration.

## Identity and routing

- Project membership is the authorization boundary for HTTP and MCP. Workspace
  administration never grants access to sibling projects or project files.
- Clerk owns sign-in, verification and OAuth consent. Do not build a second identity
  system or trust an incoming Host header to advertise OAuth/MCP service URLs.
- OAuth access requires trusted Tin resource binding from Clerk introspection or a
  signature-verified access JWT. `TIN_LITE_MCP_OAUTH_CLIENT_IDS` is an optional explicit
  legacy policy for unbound tokens, not a new-user enrollment step. Never admit clients
  by name, DCR success or membership alone, or override conflicting signed claims.
  Keep MCP and connection setup on the shared verifier; browser sessions stay separate.
- Dashboard routes are ordinary paths: /system, /chat, /activity, /decisions, /files,
  /integrations and /billing. Use the History API and explicit FastAPI shell routes,
  not hash routing or a catch-all that intercepts APIs, MCP, webhooks or callbacks.
- TIN_LITE_PUBLIC_URL is the configured service origin. Optional TIN_LITE_APP_URL
  defaults to it. Legacy-origin compatibility is inbound only; it must never supply
  new service links. Keep issuer, memberships and validated auth returns unchanged
  during hostname changes. See [domain configuration](docs/app-domain-rollout.md).
- Onboarding and invitations are idempotent. Accept a valid project invitation before
  provisioning a personal container. Do not add hidden-project preferences; deletion
  uses the existing creator-authorized, confirmation-bound service and keeps billing history.

## Durable execution

- Temporal histories carry identifiers and small control facts, never prompts, file
  bodies, credentials, transcript contents or model output. Activities perform I/O.
- Every run pins its immutable definition/source revision, normalized inputs, review
  policy and applicable auth/rate contract. Saved configuration changes affect future
  runs only. Preserve replay compatibility and immutable historical migrations.
- Product reads come from Postgres, not Temporal or in-memory execution handles.
  Files remain retrievable through code.storage after sandbox deletion.
- Canonical writes are serialized per project and use expectedHeadSha. Validate the
  active sandbox lease, generation and fencing token before accepting results.
- Receipt external effects before dispatch. Reuse completed effects on retries;
  uncertain paid/provider attempts are not silently purchased again or counted as zero.
- Preserve retained output and later edits on publication conflicts. Applying a saved
  version requires the existing authenticated exact-change/revision contract.
- Schedules use Temporal's existing dispatcher, skip overlap, and bounded catch-up.
  Calendar support is declared by the selected workflow, not inferred from its name.

## Models, sandboxes and connections

- New Codex compute is API-only. The provider credential stays on the switchboard;
  protected controllers receive temporary run-bound relay grants. Never restore pooled
  ChatGPT auth.json, a login broker, refresh write-back or an OAuth compute fallback.
- Historical OAuth pins retain their accounting and artifacts but cannot create new
  compute. Never silently reprice or switch them to API execution.
- Tin uses TIN_LITE_LUNA_API_KEY for its trusted OpenAI adapter. Model routes explicitly
  name provider, model and capabilities. Optional provider credentials do not register
  routes automatically. Never expose reusable keys in a sandbox, log or project file.
- Keep the protected controller separate from user/agent commands. Preserve output
  bounds, result validators, worker isolation, fenced egress, TLS proxy and cleanup.
- Browser execution is a specific pinned sandbox profile, not permission for arbitrary
  private code to access the network. Rebuild and verify images when runtime contracts change.
- Project integrations enforce selected resources and bounded operations. Custom API
  connections use the trusted service gateway; author code receives a service binding,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tin-computer/tin](https://github.com/tin-computer/tin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-18 -->
