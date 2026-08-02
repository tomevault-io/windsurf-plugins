---
trigger: always_on
description: This document contains project-specific anti-patterns and style guidelines for working on pathfinding.cloud.
---

# Claude Code Guidelines for pathfinding.cloud

This document contains project-specific anti-patterns and style guidelines for working on pathfinding.cloud.

**For field definitions and validation rules**, see [SCHEMA.md](../SCHEMA.md).
**For workflow and commands**, see [CLAUDE.md](../CLAUDE.md).

## Anti-Patterns to Avoid

### 1. Inconsistent Terminology for Parent/Variant Relationships

**WRONG:** Mixing terminology inconsistently
- Using "child" in UI text
- Using "primary" in YAML field names
- Using "parent" in user-facing documentation

**CORRECT:** Use terminology appropriate to context
- **In YAML/code**: Always use `parent` field name
- **In UI/user-facing text**: Always use "Primary Technique" and "Variants"
- **In comments**: Use "primary" and "variant" for clarity

**Rationale**: We maintain this hybrid approach for semantic clarity:
- `parent` is concise and conventional in code (`parent.id`, `parent.modification`)
- "Primary Technique" conveys foundational/original technique to users
- "Variant" explains what the path IS (expanded applicability) not just hierarchy

See [Terminology section in CLAUDE.md](../CLAUDE.md#terminology-parentchild-vs-primaryvariant) for complete guidance.

### 2. Path Name Formatting

**WRONG:** `iam:PassRole+sagemaker:CreateTrainingJob`

**CORRECT:** `iam:PassRole + sagemaker:CreateTrainingJob`

Always include spaces before and after the `+` sign when combining multiple permissions in the `name` field.

### 3. PowerUserAccess is NOT Administrative Access

**WRONG:** `The role must have administrative permissions (e.g., AdministratorAccess or PowerUserAccess)`

**CORRECT:** `The role must have administrative permissions (e.g., AdministratorAccess or an equivalent custom policy)`

PowerUserAccess does NOT provide administrative permissions (it specifically excludes IAM actions). When describing administrative access requirements in prerequisites, use "AdministratorAccess or an equivalent custom policy" instead.

### 4. Description Field Line Breaks

**WRONG:**
```yaml
description: A principal with `iam:PassRole` and `ec2:RunInstances` can create
  an EC2 instance with a privileged IAM role attached.
```

**CORRECT:**
```yaml
description: A principal with `iam:PassRole` and `ec2:RunInstances` can create an EC2 instance with a privileged IAM role attached. The instance automatically assumes the passed role, and the attacker can access the instance to retrieve temporary credentials.
```

Descriptions should be single-line in YAML (no artificial line breaks at ~80 characters). They will flow naturally in the UI based on container width.

### 5. Missing Backticks for IAM Permissions

**WRONG:** `A principal with iam:PassRole and ec2:RunInstances...`

**CORRECT:** ``A principal with `iam:PassRole` and `ec2:RunInstances`...``

All IAM permissions in descriptions, recommendations, and text should be formatted with backticks for code styling. This applies everywhere EXCEPT in the `name` field (which should be plain text).

### 6. Using Legacy Permission Format

**WRONG (deprecated format):**
```yaml
requiredPermissions:
  - permission: iam:PassRole
```

**CORRECT (current format):**
```yaml
permissions:
  required:
    - permission: iam:PassRole
      resourceConstraints: Target role ARN must be in the Resource section
  additional:
    - permission: iam:ListRoles
      resourceConstraints: Helpful for discovering available roles to pass
```

The `permissions` field separates required permissions (minimum needed) from additional helpful permissions (get/list type permissions).

## YAML Formatting Standards

### Description Fields

- **Single-line**: Descriptions should be single-line in YAML (no artificial line breaks)
- **Natural flow**: Text will wrap naturally in the UI based on container width
- **Backticks**: Use backticks for inline code formatting (e.g., `` `iam:PassRole` ``)

### Multi-line Fields

Use the `|` pipe syntax for multi-line fields:
- `recommendation`
- `command` (in exploitation steps)
- `limitations`
- Long prerequisite descriptions

Example:
```yaml
recommendation: |
  Restrict the `iam:PassRole` permission using the principle of least privilege.

  Use IAM policy conditions to restrict which roles can be passed:

  ```json
  {
    "Effect": "Allow",
    "Action": "iam:PassRole",
    "Resource": "arn:aws:iam::ACCOUNT:role/SpecificRole"
  }
  ```
```

### YAML Indentation

- Use **2-space indentation** (not tabs)
- List items use `-` prefix
- Strings with special characters should be quoted

### Complete Example

```yaml
id: example-001
name: iam:PassRole + ec2:RunInstances  # Note: spaces around +
category: new-passrole
services:
  - iam
  - ec2
description: A principal with `iam:PassRole` and `ec2:RunInstances` can create an EC2 instance with a privileged IAM role attached. The instance automatically assumes the passed role, and the attacker can access the instance to retrieve temporary credentials.
prerequisites:
  admin:
    - A role must exist that trusts ec2.amazonaws.com to assume it
    - The role must have administrative permissions (e.g., AdministratorAccess or an equivalent custom policy)  # Note: NOT PowerUserAccess
  lateral:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DataDog/pathfinding.cloud](https://github.com/DataDog/pathfinding.cloud) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
