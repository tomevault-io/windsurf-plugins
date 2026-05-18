---
trigger: always_on
description: - High-performance Go ORM focusing on CRUD and list operations.
---

---
description: 
globs: *.go
alwaysApply: false
---
# Rule Name: Thing ORM Usage Guide

# Description:
# Quick reference for using the Thing ORM (github.com/burugo/thing).

## Core Concept
- High-performance Go ORM focusing on CRUD and list operations.
- Built-in caching (Redis or in-memory) automatically handles single-entity and list query caching & invalidation.
- Uses Go generics for type safety and a cleaner API.
- Supports MySQL, PostgreSQL, SQLite.

## Configuration
1.  **Create DB Adapter:**
    ```go
    import "github.com/burugo/thing/drivers/db/sqlite" // Or mysql, postgres
    dbAdapter, err := sqlite.NewSQLiteAdapter(":memory:")
    ```
2.  **Create Cache Client (Optional, defaults to in-memory):**
    ```go
    import (
        "github.com/redis/go-redis/v9"
        redisCache "github.com/burugo/thing/drivers/cache/redis"
        "github.com/burugo/thing"
    )
    // Use nil for default in-memory:
    // var cacheClient thing.CacheClient = nil
    // Or Redis:
    // rdb := redis.NewClient(...)
    // cacheClient = redisCache.NewClient(rdb)
    ```
3.  **Get ORM Instance per Model:**
    ```go
    import "github.com/burugo/thing"
    // import your models package

    // For a specific model typels.User)
    userThing, err := thing.New[*models.User](mdc:db)
    ```
    *Alternative Global Config (less flexible):* `thing.Configure(dbAdapter, cacheClient)` then `thing.Use[*Model]()`.

## Basic Usage
1.  **Model Definition:** Embed `thing.BaseModel` and use `db` tags for columns.
    ```go
    type User struct {
        thing.BaseModel // Includes ID, CreatedAt, UpdatedAt, DeletedAt
        Name string `db:"name"` // Explicitly maps Name field to 'name' column
        UserAge int // db tag omitted, defaults to 'user_age' column
    }
    // Optional: Define TableName() method if different from snake_case plural struct name.
    // func (u *User) TableName() string { return "custom_users_table" }
    ```
    **Note:** While using the `db` tag is recommended for clarity, it's technically optional. If omitted, Thing ORM defaults to converting the Go field name (CamelCase, e.g., `UserAge`) to its snake_case equivalent (`user_age`) as the database column name. Use `db:"-"` to explicitly ignore a field.

    **Example:**
    ```go
    // Example: All index types and relationships
    type User struct {
        thing.BaseModel
        Name  string `db:"name,index"`                // Single-field index
        Email string `db:"email,unique"`              // Single-field unique index
        ColA  string `db:"col_a,index:idx_ab"`        // Composite index (idx_ab)
        ColB  int    `db:"col_b,index:idx_ab"`        // Composite index (idx_ab)
        KeyA  string `db:"key_a,unique:uq_ab"`        // Composite unique index (uq_ab)
        KeyB  string `db:"key_b,unique:uq_ab"`        // Composite unique index (uq_ab)
        // Relationship example:
        Books []*Book `thing:"hasMany;fk:user_id;model:Book" db:"-"`
    }
    ```

    **Indexes are always declared in the db tag (index, unique, index:..., unique:...).**
    **The thing tag is only used for relationship declarations (hasMany, belongsTo, model, fk, etc.).**
    **Do not use thing tag for index declaration.**
2.  **CRUD Operations and Querying (using `userThing` from config step):**
    - **Create/Update:** `err := userThing.Save(&userInstance)` (creates if ID is zero, updates otherwise).
    - **Read by ID:** `foundUser, err := userThing.ByID(id)` (uses cache).
    - **Read Multiple by IDs:** `userMap, err := userThing.ByIDs([]int64{1, 2, 3}, "OptionalPreloadField")` (uses cache, returns `map[int64]YourModelType`. Preloading is optional).
    - **Delete (Hard):** `err := userThing.Delete(&userInstance)`.
    - **Soft Delete:** `err := userThing.SoftDelete(&userInstance)` (sets `deleted=true` and `updated_at`).

    - **Querying Records:**
        The `thing` ORM provides a flexible way to build and execute queries.

        - **Initiating a Query:**
            You can start a query using `Query()` with `thing.QueryParams` or by chaining methods like `Where()`, `Order()`, and `Preload()`. All query methods return a `*CachedResult[T]` instance, allowing for lazy execution.
            ```go
            // Option 1: Using QueryParams
            params := thing.QueryParams{
                Where: "age > ? AND status = ?", Args: []interface{}{25, "active"},
                Order: "name ASC",
                Preloads: []string{"Books"}, // For preloading relationships
            }
            result := userThing.Query(params)

            // Option 2: Chainable methods (can be started from userThing or chained on a CachedResult)
            // Starting from userThing:
            result = userThing.Where("age > ?", 25).Order("name ASC").Preload("Books")
            // Chaining on an existing result:
            // result = existingResult.Where("status = ?", "active")
            ```

        - **Fetching Results from `CachedResult[T]`:**
            Once you have a `CachedResult[T]`, you can fetch data in various ways:
            ```go
            // Fetch a paginated list (e.g., offset 0, limit 10)
            users, err := result.Fetch(0, 10) // Uses cache

            // Fetch the first record matching the query

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [burugo/one-mcp](https://github.com/burugo/one-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
