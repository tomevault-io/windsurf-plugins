---
trigger: always_on
description: > An identity platform for humans, machines, and agents. Rust, Axum, Postgres with row-level security. 35 crates, ~670K lines.
---

# xavyo

> An identity platform for humans, machines, and agents. Rust, Axum, Postgres with row-level security. 35 crates, ~670K lines.

For maps, not territory: `llms.txt`, `llms-full.txt`, `docs/crates/index.md`, each crate's `CRATE.md`.

---

## How we work

A handful of principles. They decide more arguments than any style guide.

**Read the code before you write the code.** Almost every pattern you need is already here. `grep` is faster than asking. Find the existing shape; match it. If the existing shape is wrong, fix it everywhere — not just in your one new file. Writing code you don't fully understand is borrowing trouble at compound interest.

**Taste is built by reading more than you write.** When two correct implementations exist, prefer the one that fits in your head.

**Plain beats clever.** A `for` loop everyone reads beats a `fold` two people do. Macros only when functions won't fit. Generics only when concrete types won't. The next reader is you in six months — and you will not remember.

**When in doubt, write less.** Code is liability. Each new line earns its place by carrying weight.

**Three lines is not duplication.** Premature abstraction costs more than premature copy-paste. Wait for the fourth occurrence — and the *shape* the fourth one wants — before you extract. Abstractions are easier to introduce than to remove.

**Make it work, make it right, make it fast — in that order.** No optimization without a measurement. No generalization without a second case. No flag without someone needing the "off" position.

**Errors are values; panics are bugs.** `thiserror` for libraries. Propagate with `?`. Surface tenant-safe messages at the HTTP edge. **No `.unwrap()` in code an HTTP request can reach.** A panic at 3am is not a hypothetical — it's a Tuesday.

**Trust internal boundaries; validate at the edges.** Inside the workspace, code talks to code you control. Validate user input at handlers; validate external responses at connectors. Stop re-checking invariants you've already enforced.

**Delete first.** No dead branches. No commented-out code. No `// TODO: fix later` without an owner and a date. `git log` remembers; your file shouldn't.

**Tight loops win.** `cargo check -p <crate>` before `cargo test`. Per-crate before workspace. Five minutes waiting on a build to learn you misspelled a field is five minutes you don't get back.

**The PR explains the why; the code explains the what.** Names do most of the work. Comments are for non-obvious *why* — hidden constraints, subtle invariants, workarounds for a specific bug. Not for narrating "added for issue #123".

---

## The rules that aren't negotiable

These three are infrastructure, not opinion. Break them and the system breaks.

### 1. No frontend code in this repo

Everything ships as a REST API. `apps/idp-web/` and `packages/ui/` are frozen — don't touch, don't extend, don't reanimate. If you're typing JSX, you're in the wrong repo.

### 2. Multi-tenancy is a database invariant — belt *and* suspenders

Every tenant-scoped query carries `tenant_id` **explicitly**, and runs under a tenant context for RLS. RLS is the safety net; the explicit `WHERE` is the seatbelt. Use both.

```rust
pub async fn handler(Extension(claims): Extension<JwtClaims>) -> Result<...> {
    let tenant_id = claims
        .tenant_id()
        .map(|t| *t.as_uuid())
        .ok_or_else(|| /* error */)?;
    // tenant_id flows into every DB call from here. No exceptions.
}
```

```sql
-- Every query, every time.
SELECT * FROM resources WHERE tenant_id = $1 AND id = $2;
UPDATE resources SET name = $3 WHERE tenant_id = $1 AND id = $2;
DELETE FROM resources       WHERE tenant_id = $1 AND id = $2;

-- JOINs: filter on BOTH sides. Cross-tenant joins are the classic RLS bypass.
SELECT u.*
FROM group_memberships gm
JOIN users u ON gm.user_id = u.id AND u.tenant_id = $1
WHERE gm.tenant_id = $1;
```

**Never:**
- `Uuid::nil()` as a `tenant_id` placeholder in code that touches the DB.
- A handler signature missing `Extension<JwtClaims>` without an explicit, justified comment.
- A new table without `tenant_id UUID NOT NULL`, `ENABLE ROW LEVEL SECURITY`, *and* a tenant isolation policy.
- Stashing `tenant_id` in shared `State`. It belongs to the request, not the process.

### 3. The pre-commit gate

```bash
cargo fmt
cargo clippy -p <crate> -- -D warnings
cargo test -p <crate>
```

If `cargo fmt --check` is failing on `master`, you fix it in your PR — not the next person's. CI is not optional politeness; it's the source of truth.

If you change a public API, update the crate's `CRATE.md` in the same commit. If you change a crate's maturity, update `docs/crates/index.md`, `docs/crates/maturity-matrix.md`, and `llms.txt`. Three places. Yes, all three.

---

## Commands

```bash
# The inner loop (fast).
cargo check   -p <crate>
cargo test    -p <crate>
cargo clippy  -p <crate> -- -D warnings

# The outer loop (slow — use sparingly).
cargo test    --workspace
cargo fmt     --check

# Dev environment.
docker compose -f docker/docker-compose.yml up -d
```

`xavyo-ext-authz` needs `protoc` (`brew install protobuf` on macOS) for its build script.

---

## Dev email — Mailpit


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xavyo/xavyo](https://github.com/xavyo/xavyo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
