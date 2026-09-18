---
trigger: always_on
description: Rust re-implementation of Mastodon.
---

Eunha
=====

Rust re-implementation of Mastodon.

Eunha aims for 100% Mastodon database schema compatibility, so that Eunha can
be a drop-in replacement on top of your existing Mastodon database.

We track the latest Mastodon release, and provide migration path from old Eunha
database schema to updated Mastodon database schema.

It's not Eunha's goal to completely mimic Mastodon's feature set or its
implementation detail, and Eunha may contain behavioral differences.

An instance may answer additional HTTP hostnames without changing its canonical
ActivityPub identity by listing `aliases` under `[instance]`. Handlers continue
to emit URLs and account identities using `instance.domain`; aliases only affect
the shared runtime's initial Host dispatch.

~~~~ toml
[instance]
domain = "garden.eunha.space"
aliases = ["garden.eunha.site"]
~~~~


Contributing
------------

All Mastodon tables should go in the `public` schema, while tables needed for
Eunha goes in the `eunha` schema.

Use mise for all tasks. See `mise.toml`.

Use [shadcn/ui] CLI when adding components. Don't hand-roll components.

[shadcn/ui]: https://ui.shadcn.com


Federation
----------

For all federation related tasks, we use [feder], and extend it when necessary.

The extension eunha designs for the places ActivityPub scales badly is recorded
in [PROTOCOL.md](./PROTOCOL.md): the dereference storm a boost sets off, the
absence of backfill, and identity that cannot outlive a hostname. It is a design
record rather than a description of what eunha does today, and it says which is
which.

[feder]: https://github.com/limeburst/feder


Shared Redis
------------

Eunha uses unprefixed Redis keys by default, which is appropriate when an
instance has a dedicated Redis process. A pooled deployment must give every
instance a unique prefix and a Redis user restricted to that prefix:

~~~~ toml
redis_url = "redis://tenant-example:password@redis-pool:6379/0"
redis_key_prefix = "tenant-example"
~~~~

The prefix may contain ASCII letters, digits, hyphens and underscores. Eunha
adds the separating colon, so the ACL key pattern for the example is
`~tenant-example:*`. Every Redis key Eunha owns — feeds, feed population
markers, ActivityPub locks and tombstones, posting idempotency, and notification
group state — uses that namespace.

Do not treat a prefix as authorization. Give each instance a distinct Redis
user, the matching key pattern, and only the commands Eunha uses:

~~~~
+get +set +setex +exists +fcall +zadd +zremrangebyrank +zrem
+zrangebyscore +zrevrangebyscore +mget +del
~~~~

The hosting provisioner installs the fixed `eunha_compare_delete` function used
for lock release. Tenant users receive `FCALL`, but not `EVAL`, `EVALSHA`,
`SCRIPT` or `FUNCTION`, so a compromised credential cannot submit arbitrary Lua
to the shared event loop. Dedicated Redis remains zero-configuration: Eunha
falls back to its existing inline script when the named function is absent.
`INFO` is optional; without it the admin API reports the Redis version as
unknown. Process-wide memory from `INFO memory` is never exposed when a key
prefix is configured. Set `redis_process_metrics = false` to suppress it for an
otherwise dedicated deployment as well.

ACLs do not isolate CPU, memory, eviction or persistence. A shared pool remains
one performance and failure boundary and needs monitoring, bounded feed
retention, admission controls, and a path for moving heavy tenants to dedicated
Redis.

Feeds and their population markers use `redis_url`; they are bounded cache
state. Set `redis_coordination_url` to route locks, ActivityPub deletion
tombstones, posting idempotency and notification grouping to a separate
non-evicting Redis pool. If it is absent, both classes use `redis_url` as they
did before this option existed. Both endpoints use the same `redis_key_prefix`
and tenant credentials may differ by embedding them in their respective URLs.
Process-wide memory is omitted from tenant-facing admin responses whenever a
prefix or separate coordination endpoint is configured.


Several instances in one process
--------------------------------

Instances may share an S3-compatible media bucket when every one has a stable,
unique object namespace. Set `media_storage.key_prefix` to prepend that
namespace to every object read, write, delete and public URL. Leave it empty
for the historical dedicated-bucket layout:

~~~~ toml
[media_storage]
bucket = "eunha-media"
key_prefix = "tenants/9bd0de00b92141828d4bd2d36222f70c"
base_url = "https://r2.eunha.space"
~~~~

The prefix is an ownership boundary for object layout, not authorization;
bucket credentials can still access other prefixes in the same bucket.

Every eunha process serves a registry of instances and hands each request to
one of them by its `Host` header. Run without arguments, it serves the single
instance in `config.toml` and the environment, as it always has, and answers
whatever host it is asked by. Given a directory, it serves one instance per
`*.toml` in it, each answering to its `instance.domain`:

~~~~
eunha --tenants /srv/eunha/tenants migrate
eunha --tenants /srv/eunha/tenants
~~~~

Each instance keeps its own database, Redis prefix, background tasks and
signing keys; what they share is the process and its routes, built once rather

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [limeburst/eunha](https://github.com/limeburst/eunha) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-18 -->
