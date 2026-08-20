---
trigger: always_on
description: > JSN v4.x — Node.js — branch: `main`
---

# Agent Documentation for JSN CLI

> JSN v4.x — Node.js — branch: `main`

This document provides guidance for AI agents using the JSN CLI to interact with ServiceNow.

## Design Philosophy

JSN is designed for **safe, composable automation**:

1. **Read-only by default** — List and get operations are safe
2. **Explicit mutations** — Create/update/delete require explicit flags
3. **Idempotent operations** — Running the same command twice produces the same result
4. **Structured output** — JSON output can be piped to other tools
5. **Error handling** — Clear error messages with hints for resolution

## Common Workflows

### Workflow 1: Incident Management

```bash
# List all open critical incidents
jsn incidents list --query "priority=1^active=true^state!=6" --json

# Get details of a specific incident
jsn incidents INC0010001 --json

# Create a new incident (returns the created record)
jsn incidents create --description "Issue description" --priority 2 --json

# Update an incident status
jsn incidents update INC0010001 --data '{"state": "2", "assigned_to": "user_id"}'

# Add a work note to an incident
jsn records update --table incident --sys-id <sys_id> --data '{"work_notes": "Updated status"}'
```

### Workflow 2: Change Request Management

```bash
# List pending changes
jsn changes list --query "state=-5" --json

# Create a standard change
jsn changes create --description "Monthly maintenance" --risk low --json

# Approve a change (move to assessment)
jsn changes update CHG0010001 --data '{"state": "1"}'
```

### Workflow 3: User Management

```bash
# Search for users
jsn users "John Smith" --json

# Get user details
jsn users list --query "user_name=john.smith" --json

# Find user's group memberships
jsn records list --table sys_user_grmember --query "user.name=john.smith" --columns "group.name,group.manager" --json
```

### Workflow 4: Development Tasks

```bash
# List script includes in a scope
jsn dev includes list --query "sys_scope.scope=x_myapp" --json

# Get script include code
jsn dev includes MyScriptInclude --json | jq -r '.script'

# List business rules on a table
jsn dev rules list --query "collection=incident^active=true" --json

# List client scripts on a table
jsn dev clientscripts list --query "table_name=incident^active=true" --json

# List UI actions on a table
jsn dev uiactions list --query "table=incident^active=true" --json

# List update sets
jsn dev updatesets list --query "state=in progress" --json

# Set current update set
jsn dev updatesets set "My Development" --json

# Export an update set to XML (raw XML to stdout, or --out <file>)
jsn dev updatesets export "My Update Set" --out my-update-set.xml

# List access controls (ACLs) for a table
jsn dev acls list --query "name=incident" --json

# Query system properties
jsn dev properties list --query "nameLIKEglide.encryption" --json
```

### Workflow 5: Records Inspect (Audit & Diagnostics)

```bash
# Show audit history for a record
jsn records inspect INC0010001 --audit

# Show business rules that fire on a record's table
jsn records inspect INC0010001 --rules

# Show running flows for a record
jsn records inspect INC0010001 --flows

# Run all diagnostics at once
jsn records inspect INC0010001 --all
```

### Workflow 6: Dev Commands with Full CRUD

Several dev commands now support create, update, and delete in addition to list/show:

```bash
# Create a new business rule
jsn dev rules create --data '{"name": "My Rule", "collection": "incident", "script": "gs.log(\"hello\");"}'

# Update a script include
jsn dev includes update <sys_id> --data '{"script": "// updated code"}'

# Delete a UI action
jsn dev uiactions delete <sys_id> --confirm

# Create a new ACL
jsn dev acls create --data '{"name": "incident", "operation": "read", "type": "record"}'
```

Supported dev CRUD tables: `sys_script_include`, `sys_script`, `sys_script_client`, `sys_ui_action`, `sys_ui_policy`, `sys_scope`, `sys_properties`, `sys_acl`, `sys_import_set`, `sys_ws_definition`, `sys_rest_message`, `sys_rest_message_fn`, `sys_roles`, `sys_user_role`, `sys_flow`, `sys_flow_trigger`, `sys_flow_action`, `sys_ui_page`, `sys_ui_module`, `sys_app`, `sys_application`, `sys_ui_section`, `sys_ui_related_list`, `sys_ui_list`, `sys_ui_view`, `sys_ui_form`, `sys_ui_script`, `sys_ui_message`, `sys_ui_policy_condition`, `sys_script_queue`, `sys_script_email_template`, `sys_script_rest_operation`, `sys_script_rest_message`, `sys_script_rest_operation_fn`, `sys_script_ws_definition`.

### Workflow 7: Data Queries

```bash
# Generic table query with jq processing
jsn records list --table incident --query "active=true^opened_at>javascript:gs.daysAgo(7)" --json | \
  jq -r '.[] | "\(.number): \(.short_description)"'

# Count records
jsn records list --table incident --query "priority=1" --json | jq 'length'

# Export to CSV (using jq)
jsn records list --table incident --limit 100 --json | \
  jq -r '.[] | [.number, .short_description, .priority, .state] | @csv'

# Fetch all fields from a record
jsn records get --table incident --sys-id <sys_id> --columns '*' --json
```

## Best Practices for Agents

### 1. Always Use --json for Automation

```bash
# Good - structured output for parsing
jsn incidents list --json | jq '.[].number'

# Avoid - parsing human-readable output

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jacebenson/jsn](https://github.com/jacebenson/jsn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
