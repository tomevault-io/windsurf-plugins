---
trigger: always_on
description: What a plugin may do, what the host promises it, and the rules that make an in-process plugin safe to
---

# The plugin contract

What a plugin may do, what the host promises it, and the rules that make an in-process plugin safe to
run. [`README.md`](README.md) beside this is the contract itself — capabilities, config fields,
manifests, versioning — and is required reading before touching `plugins/` or this package. This
file is the part that is easy to break by accident.

Every paragraph here records a measured failure and the fix that was chosen over the obvious one.
Read the ones covering whatever you are about to change. The always-loaded index is
[`CLAUDE.md`](../../CLAUDE.md).

## The boundary

**The JSON-safe rule now covers what is stored or sent, and nothing else.** Manifests, permissions, config fields and every `capabilities/` payload: no `Date`, no class instances, no functions, durations as integer milliseconds, dates as ISO-8601 strings. The reason is Postgres and the console's JSON, not a wire format, and it survives on that basis alone. Host methods are exempt and deliberately so: `host.fetch` returns a real `Response`, `host.signal` a real `AbortSignal`, `speak()` a real `ReadableStream`. `boundary.json.safe.ts` fails `tsc` over a registered payload and its registry-coverage test fails over a boundary interface classified in none of its three arrays, so neither can drift by accident.

**In plugin code, `undefined` means "not set". Never `null`.**

**An installed plugin reaches the host's SDK through a link the host makes, not through its own
`node_modules`.** A plugin under `PLUGINS_DIR` cannot walk up to the station's tree, so the host links its
copies of this package and of every non-optional peer into `<PLUGINS_DIR>/node_modules` before each
discovery (`apps/api/src/modules/plugins/plugin.peers.ts`; the reasoning is in `apps/api/CLAUDE.md` under
"Plugins, from the host side"). So a new REQUIRED peer here is a new link there with no other change, and an
optional one is not linked at all, which is why `vitest` is optional. A plugin that ships its own copy in a
`node_modules` beside it gets that copy instead, and the brand on `PluginError` and the duck-typed zod check
are what keep that working rather than what it is designed around.

## Trust and egress

**Plugins are trusted code, permanently.** They load through a plain dynamic `import()` into the host realm and can reach `process.env`, `fs`, and the pg pool. `host.fetch` protects an honest plugin from a hostile upstream and protects the operator from a careless plugin. It does not contain a hostile one, and no future version will: the subprocess option is closed, not deferred. Do not write docs, UI copy, or comments claiming otherwise, and do not reintroduce a constraint whose only justification is a move that is not happening.

**What the layer is FOR, and what it is not.** Four threats were tabled when this was decided and three are answered. A hostile UPSTREAM against an honest plugin: the response timeout, the three body bounds, the per-hop redirect re-check and the breaker. A CARELESS plugin against the operator: the per-upstream rate bucket, the allowlist, and the private-address refusal on anything reached through `network.open`. And a plugin's MISTAKE leaking into the host, which is why art URLs are `http(s)` only and why `scrubProcessEnv` deletes the secrets from `process.env` once the config snapshot has been taken — both hygiene against an accident, neither of them containment. The fourth, a HOSTILE plugin, is not answered and never will be; the enable dialog says so in plain words, and a manifest is a description of a well-behaved plugin's blast radius rather than a limit on a badly-behaved one. Keep that distinction in any copy you write: claiming the manifest constrains a plugin is the specific false sentence this section exists to prevent.

**Closing the subprocess option is what bought back a live object across the boundary.** While isolation was still on the table nothing could cross it but JSON, so audio came back as the `host.streams` protocol — opaque handles, sequence numbers, base64 chunks and an idempotent `close()` — with a drain loop in `SpeechService` reassembling it. All of that is deleted: `speak()` returns a `ReadableStream` and `host.fetch` a real `Response`. What survives of the protocol is only its BOUNDS, and they survive because they were never about isolation: a body is read outside the call that fetched it, so it needs its own idle, lifetime and size limits whoever is holding it.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [robert-dean/deadair](https://github.com/robert-dean/deadair) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
