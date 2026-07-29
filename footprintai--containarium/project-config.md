---
trigger: always_on
description: This file is a small set of durable conventions for AI assistants working
---

# Containarium — guidance for Claude / AI assistants

This file is a small set of durable conventions for AI assistants working
on this repository. Keep it short — only rules whose violation we'd
actually want to catch in review.

## No instance / endpoint / tenant names in OSS-visible files

This repository is public-facing. **Anonymize anything that names a
specific deployment** in files that ship with the repo: `docs/`,
`README.md`, `CHANGELOG.md`, design notes, runbooks, PR descriptions,
and commit messages.

| Don't write | Do write |
| --- | --- |
| Concrete backend / lab node hostnames | "a backend host" / `<host>` / "the GPU node" |
| Live production cluster apex hostnames | "the cluster's apex hostname" / `<cluster>.example.com` |
| Tenant container names (`<tenant>-container` with real tenants) | "a tenant container" / `<tenant>-container` (generic) |
| Containarium-core service container names (in operator-specific context) | "the platform Postgres LXC" / "core service LXCs" |
| Live IPs (LAN, GCP, Tailscale) | "a private LAN IP" / `<sentinel-ip>` |
| Internal tenant workload names (docker container, app names) | "a tenant workload" / "a docker compose service" |

**Why:** concrete names are reconnaissance signals — they reveal what's
running where, which workloads to target, which backend names to
fingerprint. The cost of generic wording is one extra word; the cost of
leaking is permanent — PR descriptions and merged commits stay indexed
forever.

**Where concrete names ARE fine:**
- Private operator runbooks not in the repo.
- Internal Slack / email / private docs.
- Operator memory files outside the repo.

**Before pushing any doc / runbook / PR description:** grep the diff
for concrete names; if you're not sure, anonymize.

## CLI-first, MCP wraps it

When adding a new platform action (anything that mutates Containarium
state — create, expose, route, deploy, etc.):

1. Land it as a `containarium <verb>` cobra subcommand under
   `internal/cmd/<verb>.go` first.
2. The MCP tool in `internal/mcp/tools.go` is a **thin wrapper** over the
   same underlying Go function used by the CLI handler. Don't have the
   MCP tool call an HTTP endpoint that the CLI doesn't.

**Why:** The CLI is the canonical interface. Humans, shell scripts, CI,
and demo recordings all consume it; MCP is one specific consumer (AI
agents). Building CLI-first means:

- Anything an agent can do, a human can do via shell — symmetric surface
  with no agent-only escape hatches.
- Demo recordings are reproducible `bash` scripts, not "spin up an
  agent + JWT token" rituals.
- Tests focus on the CLI handler / shared client function; MCP correctness
  follows for free.
- The OSS community gets value from the CLI even without running an agent.

**Anti-pattern:** an MCP tool that talks to an HTTP endpoint with no
matching `containarium <verb>` subcommand. If you spot one, file a
follow-up to add the CLI counterpart.

## API: protobuf contract first, grpc-gateway for REST

The API is defined in `proto/containarium/v1/*.proto` *first*. Everything
else — gRPC server stubs, the HTTP/REST shim, the OpenAPI swagger doc,
the typed client — is **generated** from those protos via `make proto`
(which runs `buf generate`).

When adding a new endpoint:

1. Add the RPC + request/response messages in `.proto`.
2. Annotate the RPC with `(google.api.http)` for the REST verb+path
   mapping and `(grpc.gateway.protoc_gen_openapiv2.options.openapiv2_operation)`
   for the swagger description.
3. `make proto` to regenerate `pkg/pb/`, the `.pb.gw.go` gateway shim,
   and `api/swagger/containarium.swagger.json`.
4. Implement the gRPC method in `internal/server/`.
5. Wire the typed client method in `internal/client/{grpc.go, http.go}`.

**Why:** one contract drives three consumers (gRPC clients, REST/HTTP
clients via grpc-gateway, and the OpenAPI viewer) — they cannot drift
because they all regenerate from the same source. The MCP server
(which speaks REST through grpc-gateway) gets every new endpoint for
free. The CLI adds a thin cobra subcommand that calls the generated
client.

**Anti-pattern:** writing a hand-rolled `net/http` handler under
`internal/gateway/` for a new customer-facing endpoint. A handful of
legacy or internal-only endpoints (e.g. `/healthz`,
`/authorized-keys/sentinel`) live in the gateway directly — those
predate the convention or are infrastructure plumbing not in the
product contract. For anything an external caller, the CLI, or the
MCP server should hit, go through proto.

## Strong typing — use the type system, not strings

A corollary of proto-first: when proto already gives us typed
primitives, use them.

- **Protobuf enums over magic strings.** If a field's value is "must
  be one of X, Y, Z," it's an enum. Define the enum in `.proto`,
  regenerate, and let the Go code use typed constants. Example: an
  `os_type` field that accepts `ubuntu | rocky9 | rhel9` becomes a
  `OSType` enum on the proto and `pb.OSType_*` constants in Go —
  not a `string` parameter with a comment listing the allowed values.

- **Well-defined Go structs over `map[string]interface{}`.** Every
  wire payload deserves a named struct with explicit fields. The only
  legitimate uses of `map[string]interface{}` are at the type-erasing

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FootprintAI/Containarium](https://github.com/FootprintAI/Containarium) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
