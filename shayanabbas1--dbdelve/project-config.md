---
trigger: always_on
description: DBDelve is a native database client in Rust on GPUI for macOS, Linux and Windows,
---

# AGENTS.md

DBDelve is a native database client in Rust on GPUI for macOS, Linux and Windows,
speaking Postgres, MySQL, SQLite, Snowflake and SQL Server.

The split everything below leans on is _whose SQL it is_. An editor buffer is
the user's and is never touched uninvited; a browsing surface (an object tab's
preview) runs SQL DBDelve generates, regenerated from visible controls and
inspectable, never spliced into anyone's buffer.

**Read this file before doing anything.** It is the source of truth for how
DBDelve is built and why.

## Hard rules

Violating one of these is a bug regardless of the benefit. If a task seems to
require it, stop and raise it instead.

1. **Never rewrite SQL behind the user's back.** No silent `LIMIT` injection, no
   column projection, no reformatting on execute, and nothing at all on a
   statement the user did not ask DBDelve to change. A client that silently
   alters statements cannot be trusted with the statements that matter, which
   is why "silently" is the word that carries the rule. Row limits apply to
   DBDelve-generated preview queries only, and they are visible in the UI.

   DBDelve _does_ write SQL when the user asks it to, and only then, always
   where the user can read it:

   - A header click asking for a sort splices the `ORDER BY` into the
     statement in the buffer, where it can be read, edited and undone; the
     statement that runs is the statement on screen.
   - Grid edits on a query tab are appended to the buffer
     (`sql::appended_statement`) and run from there. On an object tab, which
     has no buffer, the statement is shown in a review dialog before it runs.
   - **Explain** puts the engine's `EXPLAIN` prefix on a copy of the
     statement, never into the buffer.
   - **Format Query** rewrites the buffer on command only, through
     `sqlformat`'s token-level reformatter. Never an AST round-trip: that
     regenerates the statement and drops every comment the user wrote.

   Limits on what DBDelve may write. It never writes `DROP` or `TRUNCATE`,
   whatever the user asked for. It writes `DELETE` only as the explicit
   deletion of one named row: by primary key, from a direct ask, with the
   statement shown before it runs (`sql::delete_row`). And it never writes into
   a statement it cannot parse whole: `sql::with_order_by` refuses rather than
   guessing at a clause boundary, because a corrupted statement is worse than
   an unsorted grid.

2. **Generated SQL passes a whitelist gate, and there is one per path.**
   `sql::is_generated_write` is the single gate every statement the grid writes
   passes first. It admits exactly three shapes:

   - a batch of `UPDATE`s, optionally bracketed by a `BEGIN`/`COMMIT` the gate
     can see closed;
   - one `INSERT`, naming the columns it fills;
   - one `DELETE` whose `WHERE` is a conjunction of equality predicates over
     distinct, unqualified columns against single-quoted literals (or a bare
     `0x…` hex literal, SQL Server's spelling of bytes): no `OR`, no
     other operator, no subquery, no function call, no CTE beside it, no
     `RETURNING`, no `LIMIT`, and nothing else in the submission.

   Because it is a whitelist, `DROP` and `TRUNCATE` are refused structurally,
   anywhere in the tree, CTEs included (`sql::forbidden`), and so is every
   `delete` outside that one shape (`sql::deletes_anything`, checked on the
   `INSERT` and `UPDATE` arms).

   `sql::is_generated_select` guards the filter bar, the one place user text is
   spliced into DBDelve's statement: exactly one query, nothing destructive
   under it, no delete at all. It admits no write and is not a way around the
   first gate. Do not add a path that bypasses either.

   **The `DELETE`'s shape is verified from the parse tree, not trusted because
   `sql::delete_row` produced it.** A gate that trusts its caller is a comment;
   the check lives in the gate rather than the generator precisely so the two
   can disagree. `sql::delete_matches_key` answers the half the gate cannot,
   whether the `WHERE` names exactly the row's key as a set. It is a readout,
   not a second gate: it admits nothing, and a caller runs both.

   **Multi-row deletion is not admitted.** When it is wanted, the path is the
   `BEGIN`/`COMMIT` bracketing multi-row edits already use: one `DELETE` per
   row, each naming its own key, never one predicate covering several.

   A cell is editable only when DBDelve can name its row by primary key; when
   it cannot, the grid stays read-only and says why, and never guesses at a
   predicate. An `INSERT` has no existing row to name, so a table without a
   primary key can be inserted into and not edited. That asymmetry is
   deliberate and belongs in anything that documents either feature.

3. **No environment-specific behaviour.** No vendor binary names in error
   strings, no assumption that a loopback host means plaintext, no hardcoded
   ports or hostnames. DBDelve is a generic client.

4. **Driver types do not reach the UI layer.** The grid receives rendered
   strings and type tags, never a `postgres::Row`, a `mysql::Value`, a
   `rusqlite::ValueRef`, a `tiberius::ColumnData`, an OID, a storage class or

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ShayanAbbas1/dbdelve](https://github.com/ShayanAbbas1/dbdelve) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
