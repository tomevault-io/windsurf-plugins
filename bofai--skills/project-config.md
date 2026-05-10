---
trigger: always_on
description: This skill requires:
---

# Agent Guidelines: Skills Repository

This document provides essential information for AI agents working on skills in this repository.

---

## 📁 Repository Structure

```
skills/
├── AGENTS.md                    # This file - development guidelines
├── sunswap/                     # SunSwap DEX trading skill
│   ├── SKILL.md                 # Main skill definition
│   ├── examples/                # Usage examples
│   ├── resources/               # Configuration files
│   └── scripts/                 # Helper scripts
└── [other-skills]/              # Future skills
```

---

## 🎯 What is a Skill?

A **skill** is a reusable capability that AI agents can use to accomplish specific tasks. Each skill:

- **Encapsulates domain knowledge** (e.g., how to use SunSwap DEX)
- **Provides step-by-step instructions** for AI agents
- **Includes examples** showing common usage patterns
- **May depend on external tools** (e.g., MCP servers)

---

## 📝 Skill File Format

### SKILL.md Structure

Every skill must have a `SKILL.md` file with this format:

```markdown
---
name: Skill Name
description: Brief description of what the skill does
version: 1.0.0
dependencies:
  - dependency-1
  - dependency-2
tags:
  - tag1
  - tag2
---

# Skill Name

## Overview
[What this skill does]

## Prerequisites
[What needs to be set up before using this skill]

## Usage Instructions
[Step-by-step guide for AI agents]

## Examples
[Links to example files or inline examples]

## Error Handling
[Common errors and how to handle them]

## Security Considerations
[Important security notes]
```

### YAML Frontmatter Fields

| Field | Required | Description |
|-------|----------|-------------|
| `name` | ✅ | Human-readable skill name |
| `description` | ✅ | Brief description (1-2 sentences) |
| `version` | ✅ | Semantic version (e.g., 1.0.0) |
| `dependencies` | ⚠️ | List of required tools/servers (e.g., mcp-server-tron) |
| `tags` | ⚠️ | Searchable tags for skill discovery |

---

## 🗂 Skill Directory Structure

### Required Files

- **SKILL.md** - Main skill definition (required)

### Optional Directories

- **examples/** - Usage examples (recommended)
  - Use `.md` files for documentation
  - Include complete, runnable examples
  
- **resources/** - Configuration files (optional)
  - JSON files for addresses, ABIs, constants
  - Should be referenced in SKILL.md
  
- **scripts/** - Helper scripts (optional)
  - Python, JavaScript, or shell scripts
  - Should be documented in SKILL.md

---

## 🛠 Creating a New Skill

### Step 1: Create Directory

```bash
mkdir -p skills/my-skill/{examples,resources,scripts}
```

### Step 2: Create SKILL.md

```bash
cat > skills/my-skill/SKILL.md << 'EOF'
---
name: My Skill
description: What this skill does
version: 1.0.0
tags:
  - category
---

# My Skill

## Overview
[Description]

## Usage Instructions
1. Step 1
2. Step 2
3. Step 3
EOF
```

### Step 3: Add Examples

```bash
cat > skills/my-skill/examples/basic_usage.md << 'EOF'
# Basic Usage Example

## Scenario
[What this example demonstrates]

## Steps
1. [Step 1 with code]
2. [Step 2 with code]
EOF
```

### Step 4: Test the Skill

- Read SKILL.md as an AI agent would
- Follow the instructions step-by-step
- Verify examples work as expected

---

## 📚 Writing Effective Skills

### Best Practices

#### 1. **Be Specific and Actionable**
❌ Bad: "Use the tool to get data"
✅ Good: "Call `read_contract` with contractAddress='TXX...' and functionName='balanceOf'"

#### 2. **Include Complete Examples**
- Show full tool calls with all parameters
- Include expected outputs
- Cover error cases

#### 3. **Document Dependencies Clearly**
```markdown
## Prerequisites

This skill requires:
- `mcp-server-tron` configured in your MCP client
- TRON wallet with private key set in environment
- Testnet TRX for gas fees
```

#### 4. **Use Code Blocks for Tool Calls**
```markdown
## Step 1: Get Price Quote

Use the `read_contract` tool:

\`\`\`json
{
  "contractAddress": "TKzxdSv2FZKQrEqkKVgp5DcwEXBEKMg2Ax",
  "functionName": "getAmountsOut",
  "args": [100000000, ["TUSDT...", "TWTRX..."]],
  "network": "mainnet"
}
\`\`\`
```

#### 5. **Handle Errors Gracefully**
```markdown
## Common Errors

### Error: "Insufficient allowance"
**Cause**: Token not approved for Router
**Solution**: Call `approve` function first (see example)

### Error: "Slippage too high"
**Cause**: Price moved beyond acceptable range
**Solution**: Increase slippage tolerance or retry
```

---

## 🔍 Skill Discovery

AI agents discover skills by:
1. Reading `SKILL.md` files in the skills directory
2. Matching tags to user requests
3. Following instructions in the skill

### Tagging Guidelines

Use descriptive, searchable tags:

```yaml
tags:
  - defi          # Category
  - dex           # Subcategory
  - swap          # Action
  - tron          # Blockchain
  - sunswap       # Specific protocol
```

---

## 🧪 Testing Skills

### Manual Testing Checklist

- [ ] SKILL.md has valid YAML frontmatter
- [ ] All dependencies are documented
- [ ] Instructions are clear and step-by-step
- [ ] Examples run without errors
- [ ] Resource files are valid (JSON, etc.)
- [ ] Scripts execute successfully

### Testing with AI Agents

1. **Load the skill** in your AI agent environment
2. **Ask a relevant question** (e.g., "How do I swap tokens on SunSwap?")

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BofAI/skills](https://github.com/BofAI/skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
