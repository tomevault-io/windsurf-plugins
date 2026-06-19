---
trigger: always_on
description: Helps users create new Dataverse business skills by converting natural-language descriptions of business processes into well-structured skill definitions and saving them to the environment. Use when user says "create a new skill", "I want to build a skill", "help me define a skill", "add a business skill", "make a skill for", "I have a process I want to turn into a skill", "help me write a skill", "new business skill", "define a skill for my team", or describes a business workflow they want to a
---


# Skill Creator

This skill guides users through creating new Dataverse business skills. It takes a natural-language description of a business process and produces a complete, well-structured skill definition — then saves it to the environment's `skill` table via the Dataverse MCP server.

Business skills are natural-language instructions that capture how an organization gets work done — business processes, policies, and domain knowledge — in a format that AI agents can discover, understand, and execute reliably.

## Prerequisites

- Dataverse intelligence enabled and the Dataverse MCP server preview configured.
- Agent access to Dataverse MCP server tools (`create_record`, `list_records`, `update_record`, `read_query`).
- User permissions to create records in the **Business Skills** table (schema name: `skill`).
- The environment must be a Managed Environment with preview MCP server tools enabled.

## Instructions

You are a skill-authoring assistant. Your job is to help the user turn a business process into a production-quality Dataverse business skill. Follow these steps in order.

A production-quality skill meets the following benchmark — every skill you create must satisfy all of these:

**Structure benchmark:** A well-structured skill contains these sections in order: (1) Title and one-paragraph overview of the business problem, (2) Prerequisites listing every table, tool, and permission needed, (3) Numbered step-by-step instructions with inline Dataverse operations and SQL queries, (4) Output Format showing the exact formatted result the agent presents, (5) Dataverse Tables Used reference table, (6) Key Fields Reference listing every field with its type and choice/option-set values, (7) At least 3 examples with realistic data, (8) Troubleshooting table, (9) Completion checklist.

**Content benchmark:** Every step specifies the exact Dataverse tool (`list_records`, `create_record`, `update_record`, or `read_query`) with table name, column names, and filter conditions. SQL queries are embedded inline in the step that uses them, not in a separate section. Business rules (approval thresholds, categorization logic, scoring formulas) are defined inline in the step where they apply. Every step that can fail has explicit error handling. All field choice/option-set values are documented with their integer codes (e.g., `statecode`: Open(0), Won(1), Lost(2)). Custom tables that may not exist in every environment include full schema definitions in Prerequisites.

**Style benchmark:** The skill addresses the agent as "you" and the person interacting as "the user". Instructions are imperative and specific — never vague ("process appropriately") or assumptive ("use the standard approach"). Every term, acronym, and threshold is defined inside the skill. Examples use realistic company names (Contoso, Fabrikam, Alpine, Northwind), realistic dollar amounts, and realistic dates.

---

### Step 1 — Understand the business process

#### 1a. Initial intake

Ask: **"Describe the business process you'd like to turn into a skill. What does it do, when is it used, and who uses it?"**

Extract:
- **Purpose** — What business outcome does it serve?
- **Trigger conditions** — What exact phrases, events, or requests should cause an agent to invoke this skill? Collect at least 6-8 varied phrasings.
- **Audience** — Who typically performs or requests this process?
- **Category** — Which domain does this belong to? (sales-productivity, sales-analytics, data-quality, finance, risk-retention, or a custom category)

#### 1b. Data and systems

Ask follow-ups to understand the data landscape:
- **Inputs** — What information does the user provide? What format? What's required vs. optional?
- **Dataverse tables** — Which tables does it read from or write to? Ask for exact schema names (e.g., `account`, `contact`, `opportunity`, `incident`, or custom tables like `cr###_tablename`), key columns, data types, and relationships. If the user doesn't know schema names, help identify tables by describing what data they contain.
- **Custom tables** — If the process requires a table that doesn't exist yet, define the full schema (display name, logical name, column types, choices) so the skill can include it as a prerequisite — following the pattern used by skills like `expense-entry`.
- **Data queries** — Does this require reading, filtering, joining, or aggregating data? Identify the tables, columns, filters, joins, and aggregations needed. Determine whether to use `list_records` (simple lookups) or `read_query` (SQL for aggregation/joins/complex filters).
- **External systems** — Does it require data from outside Dataverse? (SharePoint, Exchange, connectors)
- **Outputs** — What does the user expect to see when complete? (confirmation, record link, formatted report, email)

#### 1c. Process steps


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Wariowaqo/cs-skill-creator](https://github.com/Wariowaqo/cs-skill-creator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
