---
trigger: always_on
description: English | [中文](AGENTS.zh.md)
---

# AGENTS.md

English | [中文](AGENTS.zh.md)

How to work in this repository. Each rule is here because breaking it cost
something; [docs/sandbox-pitfalls.md](docs/sandbox-pitfalls.md) has the
receipts.

## DSH is a dependency, and stays one

**Never patch, vendor, or fork the harness.** It arrives from npm at the version
pinned by `DSH_VERSION` in the `Dockerfile`, and what a tenant runs is the
`lib/bin.js` the registry publishes. A change that only works against a modified
harness is not a change this project can ship.

Upgrading is a version bump, a rebuild, and an acceptance run — in that order,
and the acceptance run is not optional. The harness surfaces this project
depends on (`window.__DSH_BOOT__`, `/plugins`, the loopback-pinned configuration
methods) are not versioned APIs, so an upgrade is only known-good once the suite
says so.

If the harness genuinely cannot do what is needed, the answer is an upstream
issue and a documented limitation here — not a patch layer that silently forks.

**There is exactly one exception, and it is `web/patch-loopback.mjs`.** DSH
decides whether the settings plane is reachable from `location.hostname`, so
every tenant of a deployment reached by a domain name keeps no preference at
all — not the theme, not the language, not the conversation settings. The lock
is deliberate upstream and correct there: `trustedHosts` is a DNS-rebinding
fence, not authentication, so the configuration plane stays loopback-only until
a real authentication layer exists. This deployment is that layer, and the
tunnel already makes the server accept these writes; only the browser declines
to send them. Configuration cannot express it, composition cannot reorder around
it, and flipping the flag from a plugin lands after `ui-theme` has already
bound. The script carries the full argument and the evidence for each of those.

Two things keep it from becoming a habit. It fails the image build when it stops
matching, rather than letting a release ship with settings quietly back in
memory; and `scripts/check-images.sh` asserts it against the bytes nginx will
serve. **Add nothing beside it.** A second patch is a sign this project has
started forking the harness, which is the thing the rule above exists to
prevent — and the first one is only here because upstream is closed to it.

## Everything added to DSH is a cordis plugin

Which plugin a change belongs in is decided by one question: **take the
gateway away — is this still needed?** Five plugins sit on that question
today:

- `dsh-gateway-tunnel` carries a sandbox's `/api` traffic out to the gateway.
  It follows the transport.
- `dsh-sandbox-host` supplies what a browser needs when the backend is on a
 machine the person cannot reach: the `/files` upload channel, the settings
 document read back instead of handed to a desktop that is not there, and the
 `/browser` channel that watches the sandbox's own headless browser. Every
 line of it survives the gateway's removal, which is why it is not more surface
 on another plugin — and why it would be usable by anyone running dsh remotely.
- `dsh-tenant-account` is who is signed in, how to sign out, and the onboarding
  steps a deployment with its own sign-in page has already said. None of it
  means anything without the gateway.
- `dsh-artifact-panel` is the workspace beside the conversation — files,
  viewers, a terminal and a canvas.
- `dsh-brand` is this deployment's marks inside the shell.

The other three packages in `packages/` are not plugins: `dsh-icons` and
`dsh-ground` serve surfaces that have no module table, and `tunnel-protocol`
is the frame both ends of the tunnel speak.

A change that fits none of the five is a sign the question above has a new
answer, not that one of them should grow a second subject —
`dsh-gateway-logout` was renamed when it had three.

Four rules, each of which has broken:

- **Name plugins, never paths.** `cordis.patch.yml` refers to a plugin by
  package name. The client-module registry resolves a plugin's `package.json`
  from the config tree's baseUrl and scans only what it can resolve by name — a
  path-loaded plugin mounts its host half and contributes **no client half at
  all**, silently.
- **Install into the profile, not into `/app`.** Node resolves a plugin's own
  dependencies by walking up from where the plugin is, which never reaches
  `/app/node_modules`.
- **Use `--install-links`.** `npm install <local path>` symlinks back to the
  source, and Node then resolves the plugin's dependencies from the link target
  rather than from the profile.
- **Depend on siblings.** A plugin's dependency on another package in
  `packages/` is `file:../<name>`. Deeper relative paths only hold if every
  image copy reproduces the tree's depth, and one did not.

None of these fail the build. All of them fail on the first `import`, which is
what `scripts/check-images.sh` exists to catch.

## Do not implement somebody else's protocol

**If an official client exists for a wire protocol, use it.** envd — the
daemon every sandbox platform in this family embeds — is spoken through its
official client. Do not reimplement the wire protocol. If the official client
cannot do what is needed, the answer is an upstream issue and a documented
limitation here.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HuChundong/HamsterHQ](https://github.com/HuChundong/HamsterHQ) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
