---
trigger: always_on
description: This skill enables AI coding agents to use, extend, and test the
---

# Strata — AI Agent Skill

## Purpose

This skill enables AI coding agents to use, extend, and test the
`github.com/AndrewDonelson/strata` package correctly and efficiently.
Strata is a **four-tier auto-caching data library for Go** that unifies
in-memory cache (L1), Redis (L2), PostgreSQL (L3), and an optional
peer-to-peer distributed ledger (L4) behind a single API.

---

## Package identity

```
import "github.com/AndrewDonelson/strata"
```

Minimum requirements: Go 1.21+, PostgreSQL 14+, Redis 6+.
The package is `package strata`.
Internal sub-packages (`internal/l1`, `internal/l2`, `internal/l3`,
`internal/l4`, `internal/codec`, `internal/clock`, `internal/metrics`)
are **not** part of the public API — never import them in application code.
(Exception: `internal/l4` may be imported directly when you need the
distributed sync layer; see the L4 section below.)

---

## Read tier behaviour

```
Get(ctx, schema, id, &dest)
  │
  ├─► L1 hit?  → return immediately           (~100 ns)
  ├─► L2 hit?  → populate L1 → return         (~500 µs)
  └─► L3 hit?  → populate L2, populate L1 → return  (~5 ms)
               └─► not found → ErrNotFound
```

Writes flow in the **opposite** direction determined by `WriteMode`.

---

## Lifecycle — exactly four steps

```go
// 1. Create
ds, err := strata.NewDataStore(strata.Config{
    PostgresDSN: "postgres://user:pass@host:5432/mydb",
    RedisAddr:   "localhost:6379",
})
if err != nil { /* handle */ }
defer ds.Close() // ← always defer Close; it flushes write-behind and stops workers

// 2. Register (once per schema, at startup, before any data operation)
err = ds.Register(strata.Schema{Name: "players", Model: &Player{}})

// 3. Migrate (idempotent — safe to call on every startup)
err = ds.Migrate(ctx)

// 4. Use
_ = ds.Set(ctx, "players", "p1", &Player{...})
```

---

## Struct definition rules

- Every model **must** have exactly one field tagged `strata:"primary_key"`.
  It must be of type `string`. The field name is conventionally `ID`.
- The model **must** be a concrete struct — not a map, interface, or slice.
- Pass a **pointer** to the struct as `Schema.Model` and as `dest` in `Get`.
- The `Name` field in `Schema` defaults to the snake_case struct name if omitted.

```go
type Player struct {
    ID        string    `strata:"primary_key"`
    Username  string    `strata:"unique,index"`
    Email     string    `strata:"index,nullable"`
    Level     int       `strata:"default:1"`
    Score     int64
    APIKey    string    `strata:"encrypted"`    // AES-256-GCM, string fields only
    Password  string    `strata:"omit_cache"`   // Postgres only; never in L1/L2
    Notes     string    `strata:"nullable"`
    CreatedAt time.Time `strata:"auto_now_add"` // set once on first write
    UpdatedAt time.Time `strata:"auto_now"`     // updated on every write
    Internal  string    `strata:"-"`            // completely ignored
}
```

### Full tag reference

| Tag | Effect |
|-----|--------|
| `primary_key` | Required. Identity field for Get/Set routing. |
| `unique` | UNIQUE constraint in Postgres. |
| `index` | Non-unique Postgres index. |
| `nullable` | Column allows NULL (default is NOT NULL). |
| `omit_cache` | Excluded from L1 **and** L2; Postgres only. |
| `omit_l1` | Excluded from L1 only; still in L2. |
| `default:X` | SQL `DEFAULT X` in the DDL. |
| `auto_now_add` | Set to `time.Now()` on first insert, never updated. |
| `auto_now` | Set to `time.Now()` on every write. |
| `encrypted` | AES-256-GCM encrypted at rest. Requires `EncryptionKey` in Config. String fields only. Stored plaintext in L1/L2 for read speed. |
| `-` | Completely ignored — not persisted, not cached. |

Multiple tags are **comma-separated** with no spaces: `strata:"unique,index,nullable"`.

### Go → Postgres type mapping

| Go | PostgreSQL |
|----|-----------|
| `string` | `TEXT` |
| `int`, `int32`, `int64` | `BIGINT` |
| `float32`, `float64` | `DOUBLE PRECISION` |
| `bool` | `BOOLEAN` |
| `time.Time` | `TIMESTAMPTZ` |
| `[]byte` | `BYTEA` |
| struct / map / slice | `JSONB` |

---

## `Schema` struct — all fields

```go
type Schema struct {
    Name      string         // table/collection name (defaults to snake_case struct name)
    Model     any            // *StructType — pointer to zero value of model
    L1        MemPolicy      // in-memory cache settings
    L2        RedisPolicy    // Redis cache settings
    L3        PostgresPolicy // Postgres persistence settings
    WriteMode WriteMode      // WriteThrough | WriteBehind | WriteThroughL1Async
    Indexes   []Index        // extra database indexes
    Hooks     SchemaHooks    // lifecycle callbacks
}

type MemPolicy struct {
    TTL        time.Duration  // 0 = use Config.DefaultL1TTL
    MaxEntries int            // 0 = use Config.L1Pool.MaxEntries; PER SHARD (256 shards)
    Eviction   EvictionPolicy // EvictLRU (default) | EvictLFU | EvictFIFO
}

type RedisPolicy struct {
    TTL       time.Duration // 0 = use Config.DefaultL2TTL
    KeyPrefix string        // optional; defaults to schema name
}

type PostgresPolicy struct {
    TableName   string // optional; defaults to schema name
    ReadReplica string // optional DSN for a read-only replica
    PartitionBy string // optional column for table partitioning
}


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AndrewDonelson/Strata](https://github.com/AndrewDonelson/Strata) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
