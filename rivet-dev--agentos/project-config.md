---
trigger: always_on
description: Always spell the product name `agentOS`, never `AgentOS`; do not alter type
---

# agentOS

Always spell the product name `agentOS`, never `AgentOS`; do not alter type
identifiers such as `AgentOSActorConfig`.

agentOS owns the runtime, kernel, VFS, language execution, registry packages,
ACP/session layer, agentOS client APIs, docs, and publish machinery. agentOS
Exec is the JavaScript, TypeScript, and Python execution surface of agentOS.

## Boundaries

- Keep agentOS product versions pinned at `0.0.1` in committed files. Release
  workflows apply real versions transiently with `scripts/publish`; never commit
  release-version rewrites.
- agentOS-owned npm packages must use the `@rivet-dev/agentos-*` namespace.
  Registry software packages must use `@agentos-software/*`. Never introduce
  packages under `@agentos/*`.
- Call guest environments VMs, not sandboxes, except when referring to a package
  or public API that already uses the word.
- The protocol has no backward compatibility guarantee. Client, sidecar, and
  protocol crates ship in same-version lockstep; update both sides together.
- Generic runtime and language-execution work belongs here. Do not add a
  compatibility mirror or a second package namespace for AgentOS language execution.
- Keep root `package.json` scripts limited to Turbo orchestration; repo-specific
  commands belong in `justfile` recipes or scoped package scripts.
- agentOS targets native Linux/container execution. Browser support is not
  needed or supported here: browser sources may remain as dormant reference
  code, but their entrypoints must stay disabled and they must not enter default
  builds, CI, publication, or behavioral-parity requirements without a
  separately approved design.

## Security Model

Trust model:

- **Client**: trusted, except for code/payloads it submits for execution.
- **Sidecar/runtime**: trusted enforcement point. It owns the kernel, VFS,
  mounts/plugins, socket table, permissions, and resource policy.
- **Executor**: untrusted V8 isolate or WASM guest. Assume guest JS/Python/WASM
  and third-party packages are hostile.

The security boundary is sidecar/runtime to executor. Client-provided config is
trusted input; a guest bypassing an applied policy is in scope, while a client
choosing dangerous credentials, endpoints, mounts, or allowlists is not a
runtime escape.

The default security posture must behave like a Docker container with no host
mounts and no published ports: software inside the VM can use its virtual
filesystem, inspect its virtual process environment, spawn guest subprocesses,
bind and listen on guest sockets, and communicate over guest loopback. None of
those operations may expose a host resource by themselves. Host files enter
only through explicit mounts or copied files; host environment values enter
only through explicit configuration; guest listeners are not bound or
published on the host; and external network/DNS access requires an explicit
grant. A default network denial must block traffic that crosses the VM boundary,
not local bind, listen, or loopback traffic inside the VM. Trusted runtime
bootstrap, including launching the guest's Node interpreter, must not be
mistaken for a guest subprocess and blocked by guest `childProcess` policy.

Every limit, timeout, queue, buffer, and per-entity collection must be bounded
by default, warn near threshold, and fail with a typed error that names the
limit and how to raise it. Host-visible warnings/errors must reach stderr/log
or structured trace paths, not stay trapped in the VM.

Never swallow errors silently. Every failure must either propagate as a hard,
typed error to the caller (preferred) or be clearly logged at the failure site;
empty `catch`/`let _ =` on fallible operations and fire-and-forget promises
that drop rejections are bugs, not defensive coding. For guest-visible
surfaces, prefer matching Linux behavior — the correct POSIX errno delivered to
the guest — over inventing a softer fallback that hides the failure.

## SQLite Schema Ownership

The per-VM SQLite database is physically shared but has three independent schema
owners. Each owner must manage its own version table and append-only migration
ladder, operate only on its own table namespace, and never read, advance,
migrate, or delete another owner's schema:

- Filesystem storage owns `agentos_fs_*`, including
  `agentos_fs_schema_version`.
- Sidecar/core durable state owns `agentos_core_*`, including
  `agentos_core_schema_version`. This namespace is intentionally generic; do
  not name it after sessions, ACP, or another current consumer.
- The agentOS TypeScript actor layer owns `agentos_actor_*`, including
  `agentos_actor_schema_version`.

Do not use a shared schema-version table, a `component` discriminator, or a
global migration sequence across these owners. Each migration must update its
owner's version in the same SQLite transaction or savepoint as its schema
changes. agentOS-owned tables must be `STRICT`.

There is no compatibility requirement for the previous SQLite layout. Remove
the shared component-version mechanism and rename or replace legacy
`agentos_vfs_*`, `agentos_session*`, and `agent_os_*` tables directly; do not
add compatibility views, aliases, legacy adoption paths, or dual writes.

## Runtime And Registry


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rivet-dev/agentos](https://github.com/rivet-dev/agentos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
