---
trigger: always_on
description: This file lives at `<repository-root>/AGENTS.md` inside the `personal-context` workspace.
---

# proxbox-api Agent Index

## Workspace Context

This file lives at `<repository-root>/AGENTS.md` inside the `personal-context` workspace.
Workspace guidance: `/root/personal-context/CLAUDE.md`.
Per-repo deep-dive: `/root/personal-context/claude-reference/proxbox-api.md`.
Submodule layout and cross-repo links: `/root/personal-context/claude-reference/dependency-map.md`.

---

Use the root `CLAUDE.md` first, then open the nearest scoped guide for the code you are changing.

## Mounted Operation Inventory

Read `proxbox_api/operation_inventory/CLAUDE.md` and
`docs/operations/operation-inventory.md` before changing the offline inventory,
its explicit developer CLI, contract inputs, schemas, rendered tables, or
build-only documentation consistency hook. Preserve every ordered registration,
collision, WebSocket and generated version/alias. All twenty-two fixed feature
inputs remain required; they represent fifteen reachable states, and the
all-disabled state is unreachable. Never infer effects from HTTP methods or
equate successful generation with caller/effect readiness. Keep generated
artifacts under `contracts/`, not `docs/`, and regenerate them after changing
their complete source closure. No inventory module belongs in runtime startup.

## Proxmox Browser Console Sessions

Read [`docs/api/console-sessions.md`](docs/api/console-sessions.md) and `proxbox_api/routes/proxmox/CLAUDE.md` before changing `POST /proxmox/console/sessions`, `ConsoleSessionRequest`, `ConsoleSessionResponse`, `_request_console_proxy()`, `_console_ticket()`, `_console_port()`, `_build_ws_url()`, or `ProxmoxSession.get_websocket_auth()`. This route returns private, short-lived Proxmox transport material only to the trusted `trusted-relay-service` relay. Preserve the explicit QEMU/LXC mode matrix, local endpoint-ID meaning, stored TLS policy, full ticket encoding, exactly one API-token or password-session WebSocket authentication value, and the rule that tickets, upstream URLs, cookies, and authorization values never reach browser JavaScript or logs.

The standalone browser surface is distinct:
`POST /proxmox/console/browser-sessions` returns only `stream_token`,
`websocket_path`, `expires_at`, and `console_type`; WebSocket
`/proxmox/console/browser-stream` consumes it from exactly one
`proxbox-token.<stream_token>` offered protocol alongside `binary`; never put
the token in a URI or echo its protocol. Keep the complete private
payload Fernet-encrypted in shared SQLite, the token random and one-use, the
30-second TTL and active-count limits bounded, and consumption atomic across
workers. Require the exact validated HTTPS Origin and `binary` subprotocol.
Preserve the stored endpoint `verify_ssl` policy, mediate RFB 3.8 VNC
authentication server-side for QEMU noVNC, and keep all client errors, close
reasons, and logs secret-free. Disable ambient proxies and refuse every
upstream redirect before a second connection so credentials cannot be replayed.
QEMU terminal and LXC terminal relay frames
directly; LXC noVNC remains invalid. Both create and consume must retain the
inactive `console_relay_policy` seam for the pending RPC-only endpoint policy;
do not implement or activate that policy here. Require the current endpoint row
to be enabled at both standalone boundaries without changing the existing
service-only broker's behavior.
Preflight Fernet before requesting an upstream ticket, bracket IPv6 authorities,
validate the node path segment, validate the expiry index at startup, and use
one shared idle deadline refreshed by traffic in either direction.

Generated `/proxmox/api2/*` proxy dispatch is read-only. Every non-GET method is refused before target and credential resolution, including cached and rebuilt routes. Mutation schemas remain discoverable but deprecated with a documented 403; use dedicated typed, audited RPC procedures for supported writes. This method guard does not establish effect safety for all GET operations or change handwritten route authorization.

## Proxmox Code Generation Security

Runtime code generation defaults to disabled. Only the explicit development
setting `PROXBOX_RUNTIME_CODEGEN_ENABLED=true` mounts the HTTP generation and
route-refresh endpoints or permits runtime user-schema discovery. With the
default setting, route registration, schema discovery, and Pydantic source
rendering use bundled schemas only; the user-generated directory may receive
only the derived route cache. Refresh a bundled tag, including `latest`, solely
by installing a replacement package that bundles the updated schema.

Runtime-generated routes construct Pydantic models directly from parsed OpenAPI
data with `pydantic.create_model`; runtime startup and route refresh never
evaluate rendered Python source. The file renderer remains available for
offline artifacts, but it must validate every emitted identifier and render
aliases, descriptions, and defaults with Python literal representations.
Codegen version tags must match `^[A-Za-z0-9][A-Za-z0-9._-]{0,63}$` and must
not equal `.` or `..`. Validate tags before any crawl or write, and resolve
every generated artifact and runtime route-cache path inside its configured
base directory before accessing the filesystem.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [emersonfelipesp/proxbox-api](https://github.com/emersonfelipesp/proxbox-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-18 -->
