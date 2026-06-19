---
trigger: always_on
description: Monitor errors, manage releases, track performance, and configure alerts via the Sentry API.
---

# Sentry

Monitor errors, manage releases, track performance, and configure alerts via the Sentry API.

All commands go through `skill_exec` using CLI-style syntax.
Use `--help` at any level to discover actions and arguments.

## Projects

### List projects

```
sentry list_projects --organization my-org
```

| Argument       | Type   | Required | Description                 |
|----------------|--------|----------|-----------------------------|
| `organization` | string | yes      | Organization slug           |

Returns: list with `id`, `slug`, `name`, `platform`, `status`, `date_created`.

### Get project

```
sentry get_project --organization my-org --project backend
```

| Argument       | Type   | Required | Description       |
|----------------|--------|----------|-------------------|
| `organization` | string | yes      | Organization slug |
| `project`      | string | yes      | Project slug      |

Returns: `id`, `slug`, `name`, `platform`, `status`, `dsn`, `team`, `date_created`, `features`.

### Create project

```
sentry create_project --organization my-org --team platform --name "Backend API" --platform node
```

| Argument       | Type   | Required | Description                                |
|----------------|--------|----------|--------------------------------------------|
| `organization` | string | yes      | Organization slug                          |
| `team`         | string | yes      | Team slug                                  |
| `name`         | string | yes      | Project name                               |
| `platform`     | string | no       | Platform (e.g. `node`, `python`, `csharp`) |

Returns: `id`, `slug`, `name`, `dsn`.

## Issues

### List issues

```
sentry list_issues --organization my-org --project backend --query "is:unresolved level:error" --sort date --per_page 25
```

| Argument       | Type   | Required | Default        | Description                                 |
|----------------|--------|----------|----------------|---------------------------------------------|
| `organization` | string | yes      |                | Organization slug                           |
| `project`      | string | yes      |                | Project slug                                |
| `query`        | string | no       | `is:unresolved`| Search query (Sentry search syntax)         |
| `sort`         | string | no       | `date`         | `date`, `new`, `priority`, `freq`, `users`  |
| `per_page`     | int    | no       | 25             | Results per page (1-100)                    |
| `cursor`       | string | no       |                | Pagination cursor                           |

Returns: list with `id`, `title`, `culprit`, `level`, `status`, `count`, `user_count`, `first_seen`, `last_seen`, `permalink`.

### Get issue

```
sentry get_issue --issue_id 12345
```

| Argument   | Type   | Required | Description |
|------------|--------|----------|-------------|
| `issue_id` | string | yes      | Issue ID    |

Returns: `id`, `title`, `culprit`, `level`, `status`, `count`, `user_count`, `first_seen`, `last_seen`, `metadata`, `tags`, `assigned_to`, `permalink`.

### Resolve issue

```
sentry resolve_issue --issue_id 12345 --status resolved
```

| Argument   | Type   | Required | Default    | Description                                      |
|------------|--------|----------|------------|--------------------------------------------------|
| `issue_id` | string | yes      |            | Issue ID                                         |
| `status`   | string | no       | `resolved` | `resolved`, `resolvedInNextRelease`              |

Returns: `id`, `status`.

### Ignore issue

```
sentry ignore_issue --issue_id 12345 --ignore_count 100 --ignore_window 60
```

| Argument        | Type   | Required | Default | Description                                 |
|-----------------|--------|----------|---------|---------------------------------------------|
| `issue_id`      | string | yes      |         | Issue ID                                    |
| `ignore_count`  | int    | no       |         | Ignore until seen this many more times      |
| `ignore_window` | int    | no       |         | Time window in minutes for ignore_count     |
| `ignore_until`  | string | no       |         | ISO 8601 timestamp to ignore until          |

Returns: `id`, `status`.

### Assign issue

```
sentry assign_issue --issue_id 12345 --assignee user:jane@example.com
```

| Argument   | Type   | Required | Description                                         |
|------------|--------|----------|-----------------------------------------------------|
| `issue_id` | string | yes      | Issue ID                                            |
| `assignee` | string | yes      | `user:<email>`, `team:<slug>`, or empty to unassign |

Returns: `id`, `assigned_to`.

### Delete issue

```
sentry delete_issue --issue_id 12345
```

| Argument   | Type   | Required | Description |
|------------|--------|----------|-------------|
| `issue_id` | string | yes      | Issue ID    |

Returns: confirmation with `id`.

## Events

### List events

```
sentry list_events --organization my-org --project backend --per_page 10
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [officeos-co/skill-sentry](https://github.com/officeos-co/skill-sentry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
