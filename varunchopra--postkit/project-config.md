---
trigger: always_on
description: PostgreSQL-native auth, permissions, versioned config, usage tracking, and job queues. Pure SQL functions — works with any language or driver.
---

# Postkit — Agent Guide

PostgreSQL-native auth, permissions, versioned config, usage tracking, and job queues. Pure SQL functions — works with any language or driver.

## Setup

```bash
# 1. Clone and build
git clone https://github.com/varunchopra/postkit.git
cd postkit && make build

# 2. Install on your database (PostgreSQL 14+)
psql $DATABASE_URL -f dist/postkit.sql           # all modules
# Or individual:
psql $DATABASE_URL -f dist/authn.sql             # users, sessions, tokens
psql $DATABASE_URL -f dist/authz.sql             # permissions
psql $DATABASE_URL -f dist/config.sql            # versioned config
psql $DATABASE_URL -f dist/meter.sql             # usage metering
psql $DATABASE_URL -f dist/queue.sql             # job queues
```

For the optional Python SDK: `pip install -e "./sdk[binary]"`

## Multi-Tenancy

Postkit uses Row-Level Security for tenant isolation. Every query runs in the context of a namespace set via [`set_tenant`](docs/authn/sql.md#authnset_tenant) inside a transaction. Context is transaction-local (SET LOCAL) — it clears on commit. Without `set_tenant`, queries return nothing (fail-closed).

Functions that write data need the `p_namespace` parameter to match the tenant context. `set_tenant()` controls RLS visibility; `p_namespace` controls what gets stored.

Each module has [`set_tenant`](docs/authn/sql.md#authnset_tenant) / [`clear_tenant`](docs/authn/sql.md#authnclear_tenant). Call `clear_tenant()` before returning connections to a pool. The Python SDK handles tenant context automatically — pass `namespace` to the constructor.

## Hashing

Postkit stores hashes, never plaintext. This applies in every language.

- **Passwords:** Hash with argon2 or bcrypt. The parameter is named `p_password_hash` — see [`create_user`](docs/authn/sql.md#authncreate_user). Pass `NULL` for SSO-only users.
- **Session tokens, API keys, refresh tokens:** SHA-256. Generate a random token, hash it, store the hash in Postkit, send the raw token to the client — see [`create_session`](docs/authn/sql.md#authncreate_session).

## SQL Quick Reference

Common operations by module. All require tenant context — call `{module}.set_tenant(namespace)` in a transaction first (see Multi-Tenancy). Queue takes namespace as first parameter; other modules default to `'default'`. Each function's docs include full signatures, parameters, and usage examples — read them before integrating.

**Auth:** [`create_user`](docs/authn/sql.md#authncreate_user) · [`create_session`](docs/authn/sql.md#authncreate_session) · [`validate_session`](docs/authn/sql.md#authnvalidate_session) · [`revoke_session`](docs/authn/sql.md#authnrevoke_session) — [full reference](docs/authn/sql.md)

**Login flow:** [`is_locked_out`](docs/authn/sql.md#authnis_locked_out) → [`get_credentials`](docs/authn/sql.md#authnget_credentials) → verify hash + check `disabled_at` → [`record_login_attempt`](docs/authn/sql.md#authnrecord_login_attempt) → [`create_session`](docs/authn/sql.md#authncreate_session)

**Permissions:** [`write_tuple`](docs/authz/sql.md#authzwrite_tuple) · [`check`](docs/authz/sql.md#authzcheck) · [`add_hierarchy`](docs/authz/sql.md#authzadd_hierarchy) — [full reference](docs/authz/sql.md)

**Config:** [`set`](docs/config/sql.md#configset) · [`get`](docs/config/sql.md#configget) · [`rollback`](docs/config/sql.md#configrollback) — [full reference](docs/config/sql.md)

**Metering:** [`allocate`](docs/meter/sql.md#meterallocate) · [`reserve`](docs/meter/sql.md#meterreserve) · [`commit`](docs/meter/sql.md#metercommit) — [full reference](docs/meter/sql.md)

**Queues:** [`push`](docs/queue/sql.md#queuepush) · [`pull`](docs/queue/sql.md#queuepull) · [`ack`](docs/queue/sql.md#queueack) — [full reference](docs/queue/sql.md)

## Python SDK (Optional)

The SDK wraps every SQL function 1:1 with type hints, automatic tenant context, error mapping, and dict returns. Requires psycopg3 with the default tuple row_factory. Each function's docs include full signatures, parameters, and usage examples — read the relevant [module docs](#modules) before integrating.

**Setup:**
```python
import psycopg
from postkit.authn import AuthnClient
from postkit.authz import AuthzClient
from postkit.config import ConfigClient
from postkit.meter import MeterClient
from postkit.queue import QueueClient

conn = psycopg.connect("postgresql://localhost/myapp")
cursor = conn.cursor()  # default tuple factory — do NOT use dict_row

authn = AuthnClient(cursor, namespace="my-app")
authz = AuthzClient(cursor, namespace="my-app")
config = ConfigClient(cursor, namespace="my-app")
meter = MeterClient(cursor, namespace="my-app")
queue = QueueClient(cursor, namespace="my-app")
```

**Auth:** [`create_user`](docs/authn/sdk.md#create_user) · [`create_session`](docs/authn/sdk.md#create_session) · [`validate_session`](docs/authn/sdk.md#validate_session) · [`revoke_session`](docs/authn/sdk.md#revoke_session) — [full reference](docs/authn/sdk.md)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [varunchopra/postkit](https://github.com/varunchopra/postkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
