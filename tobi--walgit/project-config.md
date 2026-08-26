---
trigger: always_on
description: Context: **everyone, first.** Read this before touching the repository: the constraints the design answers
---

# walgit — architecture and operating manual for contributors (humans and agents)

Context: **everyone, first.** Read this before touching the repository: the constraints the design answers
(§1), how the WAL works (§2), the principles a PR is judged against (§3), every design decision in force (§4),
and the working rules (§5). Reading order starts at `GOAL.md`; §0 maps every other document.

> **No backwards compatibility (pre-1.0).** We owe nothing to previous shapes of this system. Do not keep
> aliases, fallbacks, shims, deprecated routes, old config keys, old proto fields or migration code. When a
> decision changes the shape, **delete the old shape in the same change**: routes, clients, tests, docs, config
> keys. Data in the bucket is the one exception — WAL/manifest/log formats stay append-only and replayable,
> because the bucket is the repository; everything else is disposable. If you find yourself writing "still
> accepted for …" or "legacy", stop and remove the thing instead.

walgit serves git over smart HTTP (v0/v2), receive-pack, upload-pack, bundle-uri, LFS and a browsing web UI,
written in Rust, from disposable hosts whose only durable state is an object-store bucket. The reference workload
it was built against is a 57 GiB / 73 M-object / 1.4 M-commit / 466 k-ref monorepo with LFS, served from
machines whose "disk" is 20 GiB of tmpfs, next to a long tail of small repositories. The design follows Cursor's
*Git at any scale* (Continuity); `README.md` tells the story, this file keeps the rules.

## 0. Document map (one home per fact — link, don't duplicate)

| Doc | Who / when to read it |
|---|---|
| `GOAL.md` | Everyone, first. What walgit is for, the acceptance table, what we do not optimise for. |
| `AGENTS.md` (this) | Everyone. Constraints §1, WAL §2, principles §3, decisions §4, working rules §5. |
| `README.md` | The introduction: why (the Cursor lineage), what it does, how it works briefly, running it, invariants. |
| `docs/BUNDLE_URI_DESIGN.md` | Anyone touching bundles, the scheduler, base rebuilds, or big-repo clone/fetch UX. Design of record; normative config in §4. |
| `docs/ROUNDTRIPS.md` | **Anyone touching a protocol that talks to the bucket** (publish, sync, checkpoints, compaction/leases, bundles, remote reader, store backends). Round trips are the cost model; correct is not sufficient. |
| `docs/POLICY.md` | Anyone touching receive-pack authorization or writing a repo policy. Normative rule language. |
| `docs/LFS.md` | Anyone touching LFS (`lfs.rs`, `lfs_upstream.rs`) or importing a repository whose LFS history lives elsewhere. |
| `docs/INTEGRITY.md` | Anyone touching import, the maintainer's `fsck`/`repair` units, or seeing `connectivity: missing object` on a push. |
| `docs/EVENTS.md` | Anyone changing WAL-derived ref events, the webhook bridge, consumer semantics or event cursors. |
| `docs/CONTRACT.md` | When you touch a crate boundary. The cross-crate contract; *extend, don't rename*; code wins where they differ. |
| `docs/reference/cursor-git-at-any-scale.md` | The source design, verbatim. Read once before touching WAL/publish/sync/placement. |
| `docs/patches/README.md` | Git client patches (bundle filter matching) and the gate for advertising filtered bundle families together. |
| `web/API.md` | UI/SDK authors and anyone changing `web/*.rs`. Wire contract, caching rules, SSE envelope, tasks, prefix-first lanes. |
| `web/sdk/README.md` | Users of `repos.js`. |
| `web/README.md` | Frontend engineers changing the React SPA, Vite build, SDK adapter, static assets, loading states. |
| `walgit.example.toml` | Every config key with its default and a comment. Change it with the code. |
| `walgit.standalone.toml` | The one-machine shape: `walgit-server --config walgit.standalone.toml` → `https://walgit.localhost:8080/`. |
| `deploy/nginx.conf.example` | An optional nginx in front; documents the `X-Accel-Redirect` byte-offload contract. |
| `Containerfile`, `flake.nix` | An OCI image; a Nix package, image and devshell. |

---

## 1. Constraints we design for

### 1.1 The machine: small, ephemeral, shared-nothing
- Assume an instance of **a few vCPU and tens of GiB of memory where the disk *is* that memory** (tmpfs).
  `cache.max_bytes` (default 20 GiB) is the budget for everything on disk. A monorepo's base pack alone can be
  30+ GB: **it cannot be materialized there, ever.** A host with a real SSD (`cache.mode = "disk"`) is the
  exception, placed by configuration, not the rule.
- **Instances are ephemeral and shared-nothing**: no stable identity, no node-to-node networking, no gossip.
  Explicit placement (D30) decides which hosts perform a repository's object work; refs-level reads remain
  available everywhere, including during a deploy.
- **CPU may be throttled between requests** on serverless platforms; background work there must be bounded
  (`wal.prefetch_max_bytes`).
- **Object store facts**: ~60–80 ms per GET, ~100 MB/s per connection (stripe for more), conditional GET ~15 ms;
  same-object overwrite is serialized (~1 write/s) — a single CAS'd object is a throughput cap.
- Operations that need real disk or hours of CPU (a full `repack -adb` of a monorepo, building its 30 GB weekly

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tobi/walgit](https://github.com/tobi/walgit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-24 -->
