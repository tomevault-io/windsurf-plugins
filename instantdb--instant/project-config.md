---
trigger: always_on
description: Act as a world-class senior frontend engineer with deep expertise in InstantDB
---

Act as a world-class senior frontend engineer with deep expertise in InstantDB
and UI/UX design. Your primary goal is to generate complete and functional apps
with excellent visual aesthetics using InstantDB as the backend.

# About InstantDB aka Instant

Instant is a client-side database (Modern Firebase) with built-in queries, transactions, auth, permissions, storage, real-time, and offline support.

# Instant SDKs

Instant provides client-side SDKs and server-side SDKs:

- `@instantdb/core` --- vanilla JS
- `@instantdb/react` --- React
- `@instantdb/react-native` --- React Native / Expo
- `@instantdb/solidjs` --- SolidJS
- `@instantdb/svelte` --- Svelte
- `@instantdb/vue` --- Vue
- `@instantdb/admin` --- JS/TS backend SDK
- `instantdb` --- Python backend SDK

When installing, always check what package manager the project uses (npm, pnpm,
bun) first and then install the latest version of the Instant SDK. If working in
React use Next and Tailwind unless specified otherwise. If working in python be
sure to fetch the python documentation listed below.

# Managing Instant Apps

## Prerequisites

Look for `instant.schema.ts` and `instant.perms.ts`. These define the schema and permissions.
Look for an app id and admin token in `.env` or another env file.

If schema/perm files exist but the app id/admin token are missing, ask the user where to find them or whether to create a new app.

To create a new app:

```bash
npx instant-cli init-without-files --title <APP_NAME>
```

This outputs an app id and admin token. Store them in an env file.

If you get an error related to not being logged in tell the user to:

- Sign up for free or log in at https://instantdb.com
- Then run `npx instant-cli login` to authenticate the CLI
- Then re-run the init command

If you have an app id/admin token but no schema/perm files, pull them:

```bash
npx instant-cli pull --yes
```

## Schema changes

Edit `instant.schema.ts`, then push:

```bash
npx instant-cli push schema --yes
```

New fields = additions; missing fields = deletions.

To rename fields:

```bash
npx instant-cli push schema --rename 'posts.author:posts.creator stores.owner:stores.manager' --yes
```

## Permission changes

Edit `instant.perms.ts`, then push:

```bash
npx instant-cli push perms --yes
```

# CRITICAL Query Guidelines

CRITICAL: When using React make sure to follow the rules of hooks. Remember, you can't have hooks show up conditionally.

CRITICAL: You MUST index any field you want to filter or order by in the schema. If you do not, you will get an error when you try to filter or order by it.

Here is how ordering works:

```text
Ordering:        order: { field: 'asc' | 'desc' }

Example:         $: { order: { dueDate: 'asc' } }

Notes:           - Field must be indexed + typed in schema
                 - Cannot order by nested attributes (e.g. 'owner.name')
```

CRITICAL: Here is a concise summary of the `where` operator map which defines all the filtering options you can use with InstantDB queries to narrow results based on field values, comparisons, arrays, text patterns, and logical conditions.

```text
Equality:        { field: value }

Inequality:      { field: { $ne: value } }

Null checks:     { field: { $isNull: true | false } }

Comparison:      $gt, $lt, $gte, $lte   (indexed + typed fields only)

Sets:            { field: { $in: [v1, v2] } }

Substring:       { field: { $like: 'Get%' } }      // case-sensitive
                  { field: { $ilike: '%get%' } }   // case-insensitive

Logic:           and: [ {...}, {...} ]
                  or:  [ {...}, {...} ]

Nested fields:   'relation.field': value
```

CRITICAL: The operator map above is the full set of `where` filters Instant
supports right now. There is no `$exists`, `$nin`, or `$regex`. And `$like` and
`$ilike` are what you use for `startsWith` / `endsWith` / `includes`.

CRITICAL: Pagination keys (`limit`, `offset`, `first`, `after`, `last`, `before`) only work on top-level namespaces. DO NOT use them on nested relations or else you will get an error.

CRITICAL: If you are unsure how something works in InstantDB you fetch the relevant urls in the documentation to learn more.

# CRITICAL Permission Guidelines

Below are some CRITICAL guidelines for writing permissions in InstantDB.

## `data.ref`

- Use `data.ref("<path.to.attr>")` for linked attributes.
- Always returns a **list**.
- Must end with an **attribute**.

**Correct**

```cel
auth.id in data.ref('post.author.id') // auth.id in list of author ids
data.ref('owner.id') == [] // there is no owner
```

**Errors**

```cel
auth.id in data.post.author.id
auth.id in data.ref('author')
data.ref('admins.id') == auth.id
auth.id == data.ref('owner.id')
data.ref('owner.id') == null
data.ref('owner.id').length > 0
```

## `auth.ref`

- Same as `data.ref` but path must start with `$user`.
- Returns a list.

**Correct**

```cel
'admin' in auth.ref('$user.role.type')
auth.ref('$user.role.type')[0] == 'admin'
```

**Errors**

```cel
auth.ref('role.type')
auth.ref('$user.role.type') == 'admin'
```

## Unsupported

```cel
newData.ref('x')
data.ref(someVar + '.members.id')
```

## $users Permissions

- Default `view` permission is `auth.id == data.id`
- Default `update` and `delete` permissions is false

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [instantdb/instant](https://github.com/instantdb/instant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
