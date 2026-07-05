---
trigger: always_on
description: This file provides instructions for AI agents working with the Overcut Playbooks repository. It contains essential information for creating, updating, and maintaining playbooks (workflow templates).
---

# AGENTS.md

This file provides instructions for AI agents working with the Overcut Playbooks repository. It contains essential information for creating, updating, and maintaining playbooks (workflow templates).

## 📋 Project Overview

This repository contains **Overcut Playbooks** - pre-built, customizable AI agent workflows for software development automation. Each playbook is a complete workflow that can be imported into [Overcut](https://overcut.ai) and customized for specific needs.

**Key Technologies:**

- Overcut Workflow JSON format (version 1.0.0)
- Markdown for prompts and documentation
- Git for version control

**Detailed Reference Skills:** The `.agents/skills/` directory contains deep-dive reference skills for triggers, step actions, agent sessions, template variables, prompt engineering, and agent tools. These are symlinked into each agent's config folder (`.claude/`, `.cursor/`, `.codex/`, `.gemini/`, `.agent/`) and activate automatically when relevant.

## 🏗️ Repository Structure

Each playbook must follow this structure:

```
playbook-name/
├── workflow.json          # Complete workflow definition (REQUIRED)
├── README.md              # Documentation following standard template (REQUIRED)
├── step-id-name.md        # Prompt file for each step (REQUIRED - one per step)
├── another-step-id.md     # Additional step prompts as needed
├── special-agents/        # (OPTIONAL) Specialized agent instructions
│   └── agent-name.md      # Instructions for configuring custom agents
└── ...                    # More steps as needed
```

## 🎯 Creating New Playbooks

Follow this **design-first approach** to ensure the workflow is well-planned before implementation:

### Step 1: Create Playbook Folder

- **Naming**: Use `lowercase-with-hyphens` (e.g., `auto-code-review`, `migration-assistant`)
- **Location**: Root of repository
- **Command**: `mkdir playbook-name && cd playbook-name`

### Step 2: Create README.md with High-Level Flow

**Start with the design and documentation first.** This helps clarify the workflow structure before implementation.

Use this template structure:

```markdown
# [Workflow Name]

## 📋 Overview

[2-3 sentences about what it does and outputs]

## ⚡ Triggers

[Automatic and manual triggers]

## 🎯 Use Cases

[When to use this workflow]

## 🔧 Prerequisites

[Required setup, agents needed]

## 🏗️ Workflow Steps

[Step-by-step breakdown with agent assignments and durations]

- List each step with its purpose, agent type, and estimated duration
- Describe the flow between steps
- Identify what data passes between steps

## 🎨 Customization

[How to adapt the workflow, which prompts to edit]

## 🔗 Related Workflows

[Links to related playbooks]
```

**README Requirements:**

- Complete all sections, especially the "Workflow Steps" section
- Define the high-level flow and step sequence
- Document what each step does and which agents are needed
- Include customization guide
- Document prerequisites clearly
- Add related workflow links
- Grammar and spelling checked

**Purpose**: The README serves as the design document. It should clearly describe:

- What the workflow does
- How it flows from step to step
- What each step accomplishes
- What data/context flows between steps

### Step 3: Create Step Prompt Files

**After the README is approved, create the prompt files.** This ensures the prompts align with the design and makes sense before creating the JSON structure.

**CRITICAL NAMING RULE**: Prompt filenames **MUST** match the step IDs you'll use in workflow.json (use kebab-case).

**Process:**

1. **Identify steps from README**: Based on the "Workflow Steps" section, list all steps
2. **Name each step**: Use descriptive kebab-case names (e.g., `prepare-review-plan`, `code-review`, `submit-review`)
3. **Create prompt files**: For each step, create `{step-id}.md` with detailed instructions

**Example:**

- Planned step: "Prepare Review Plan"
- Step ID: `prepare-review-plan`
- Prompt filename: `prepare-review-plan.md` ✅
- **NOT**: `step-1-prepare-review-plan.md` ❌
- **NOT**: `prepareReviewPlan.md` ❌

**For each step:**

1. Create a `.md` file with the step ID as the filename
2. Write clear, detailed instructions for that step
3. Include:
   - What the step should accomplish
   - What inputs it receives (from previous steps or triggers)
   - What outputs it should produce
   - How to handle errors
   - Format requirements for outputs

**Prompt File Guidelines:**

- Use clear, specific instructions
- Include examples when helpful
- Handle error cases explicitly
- Specify output format requirements
- Pass complete context between steps using `{{outputs.step-id.message}}` for agent steps or `{{outputs.step-id}}` for infrastructure steps
- Reference the step's role in the overall workflow
- Ensure prompts align with the README design

**Purpose**: Creating prompts first helps validate:

- The workflow design makes sense
- Steps have clear responsibilities
- Data flow between steps is well-defined
- The prompts are complete and actionable

### Step 4: Create workflow.json


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [overcut-ai/overcut-playbooks](https://github.com/overcut-ai/overcut-playbooks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-05 -->
