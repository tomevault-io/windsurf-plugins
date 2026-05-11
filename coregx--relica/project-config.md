---
trigger: always_on
description: > **Instructions for AI coding agents working with Relica.**
---

# AGENTS.md

> **Instructions for AI coding agents working with Relica.**
> **Read this BEFORE generating any code.**

---

## API Priority (MUST FOLLOW)

### 1. RECOMMENDED: Model() API for CRUD

**ALWAYS use Model() API for struct-based operations:**

```go
// INSERT - CORRECT
user := User{Name: "Alice", Email: "alice@example.com"}
err := db.Model(&user).Insert()
// user.ID is auto-populated!

// UPDATE - CORRECT
user.Name = "Alice Updated"
err := db.Model(&user).Update()

// DELETE - CORRECT
err := db.Model(&user).Delete()

// UPSERT - CORRECT (INSERT or UPDATE on conflict)
err := db.Model(&user).Upsert()              // all non-PK fields
err = db.Model(&user).Upsert("name", "email") // selective fields

// UPDATE CHANGED - CORRECT (only modified fields)
original := user
user.Name = "Updated"
err = db.Model(&user).UpdateChanged(&original)

// SELECTIVE INSERT - CORRECT
err = db.Model(&user).Insert("name", "email") // Only these fields
```

### 2. RECOMMENDED: Expression API for WHERE

**ALWAYS use typed expressions for conditions:**

```go
// CORRECT - Type-safe expressions
db.Select().From("users").
    Where(relica.Eq("status", 1)).
    Where(relica.GreaterThan("age", 18)).
    All(&users)

// CORRECT - HashExp for simple equality
db.Select().From("users").
    Where(relica.HashExp{"status": "active", "role": "admin"}).
    All(&users)

// CORRECT - Logical combinators
db.Select().From("users").
    Where(relica.And(
        relica.Eq("status", 1),
        relica.Or(
            relica.Eq("role", "admin"),
            relica.GreaterThan("age", 30),
        ),
    )).
    All(&users)

// CORRECT - IN clause
db.Select().From("users").
    Where(relica.In("status", 1, 2, 3)).
    All(&users)

// CORRECT - LIKE with escaping
db.Select().From("users").
    Where(relica.Like("name", "john")).
    All(&users)

// CORRECT - BETWEEN
db.Select().From("orders").
    Where(relica.Between("created_at", start, end)).
    All(&orders)
```

### 3. ACCEPTABLE: Named Placeholders

**Use `{:name}` syntax with `relica.Params` for readable parameterized queries:**

```go
// Named parameters - readable and safe
db.Select().From("users").
    Where("id = {:id} AND status = {:status}", relica.Params{
        "id":     userID,
        "status": "active",
    }).
    All(&users)

// Same parameter reused
db.Select().From("categories").
    Where("parent_id = {:id} OR id = {:id}", relica.Params{"id": catID}).
    All(&categories)
```

Works in `Where`, `AndWhere`, `OrWhere` on Select, Update, and Delete.

### 4. FALLBACK ONLY: Positional Placeholders

**Use ONLY when named params or expressions don't fit:**

```go
// ACCEPTABLE - Simple positional query
db.Select().From("users").
    Where("id = ?", userID).
    One(&user)

// ACCEPTABLE - Complex custom SQL
db.Select().From("users").
    Where("LOWER(email) = LOWER(?)", email).
    All(&users)
```

### 5. AVOID: map[string]interface{}

**DO NOT use map[string]interface{} for CRUD operations!**

```go
// WRONG - Don't do this!
db.Insert("users", map[string]interface{}{
    "name":  "Alice",
    "email": "alice@example.com",
}).Execute()

// CORRECT - Use Model() API instead
user := User{Name: "Alice", Email: "alice@example.com"}
db.Model(&user).Insert()
```

**map[string]interface{} is acceptable ONLY for:**
- Dynamic data from external sources (JSON API payloads)
- Schema-less operations where struct is not available
- Migration scripts with unknown column sets

---

## Query Helpers

### Exists / Count

```go
// Check existence — returns bool
exists, err := db.Select().From("users").
    Where(relica.Eq("email", email)).Exists()

// Count rows — returns int64
count, err := db.Select().From("users").
    Where(relica.Eq("status", 1)).Count()
```

### ToSQL (Query Preview)

```go
// Get SQL without executing
sql, params := db.Select().From("users").Where(relica.Eq("id", 1)).ToSQL()
// Works on Select, Update, Delete
```

### Error Handling

```go
// ErrNotFound — One() wraps sql.ErrNoRows
err := db.Select().From("users").Where(relica.Eq("id", 999)).One(&user)
if errors.Is(err, relica.ErrNotFound) { /* not found */ }

// Error classification — works with PostgreSQL, MySQL, SQLite
if relica.IsUniqueViolation(err) { /* duplicate key */ }
if relica.IsForeignKeyViolation(err) { /* FK violation */ }
if relica.IsNotNullViolation(err) { /* NOT NULL violation */ }
if relica.IsCheckViolation(err) { /* CHECK violation */ }
```

---

## Expression API Reference

### Comparison Operators

| Function | SQL | Example |
|----------|-----|---------|
| `Eq(col, val)` | `col = ?` | `Eq("status", 1)` |
| `Eq(col, nil)` | `col IS NULL` | `Eq("deleted_at", nil)` |
| `NotEq(col, val)` | `col != ?` | `NotEq("status", 0)` |
| `NotEq(col, nil)` | `col IS NOT NULL` | `NotEq("deleted_at", nil)` |
| `GreaterThan(col, val)` | `col > ?` | `GreaterThan("age", 18)` |
| `LessThan(col, val)` | `col < ?` | `LessThan("price", 100)` |
| `GreaterOrEqual(col, val)` | `col >= ?` | `GreaterOrEqual("score", 70)` |
| `LessOrEqual(col, val)` | `col <= ?` | `LessOrEqual("qty", 10)` |

### NULL Checks (IMPORTANT!)

```go
// IS NULL - use Eq with nil
db.Select().From("users").
    Where(relica.Eq("deleted_at", nil)).  // → deleted_at IS NULL
    All(&users)

// IS NOT NULL - use NotEq with nil

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [coregx/relica](https://github.com/coregx/relica) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
