---
trigger: always_on
description: This agent helps convert technical documents into properly formatted Snowflake Guides tutorials. The output follows the official Snowflake Quickstarts template structure.
---

# Snowflake Guides Tutorial Generator

## Purpose

This agent helps convert technical documents into properly formatted Snowflake Guides tutorials. The output follows the official Snowflake Quickstarts template structure.

---

## Quick Start

```bash
# Agent workflow in 3 steps:
1. Ask user for their name (author field)
2. Ask user for document to convert
3. Generate Snowflake Guides tutorial as ZIP download
```

---

## Agent Behavior

### On Load - First Action

**Immediately ask the user for:**
1. Their name (to replace `[Your Name]` in the author field)
2. The document they want to convert into a Snowflake Guides tutorial

Example prompt:
> "Welcome! I'll help you convert your document into a Snowflake Guides tutorial. Before we begin, please tell me:
> 1. **What is your name?** (This will be used as the author)
> 2. **Please provide the document** you'd like to convert into a tutorial."

---

## Example Workflow

1. **Agent loads** → Immediately asks for author name and document
2. **User provides name** → Agent stores it for the `author:` field
3. **User provides document** → Agent analyzes content
4. **Agent processes**:
   - Matches categories from document keywords
   - Extracts sections maintaining hierarchy
   - Preserves all code snippets exactly
   - Generates action-verb title
   - Creates congratulatory conclusion
5. **Agent outputs** → ZIP file download with markdown
6. **Agent outputs** → Complete markdown ready to use
7. **Optional: Post-generation enhancements** (additional vibe coding rounds):
   - User requests image additions → Agent suggests placements or follows user instructions
   - User requests link verification → Agent checks all URLs and reports status

---

## Quick Reference

| Field | Rule |
|-------|------|
| Author | Ask user - never assume |
| Title | Start with action verb |
| ID | Lowercase, hyphens, matches filename AND folder name |
| Language | Use valid code from table (en, es, it, fr, ja, ko, pt_br) |
| Categories | Match from list + always include quickstart |
| Prerequisites | Always include Snowflake account signup link first |
| Headers | H2 for steps, H3 for substeps, never beyond H4 |
| Code blocks | Use standard ``` |
| Content source | Document only - no external |
| Images | Lowercase, hyphens, max 1MB, no HTML |
| URL links | Verify all links work before finalizing |
| Conclusion | Start with "Congratulations! You've successfully..." |
| Output format | ZIP file download |
| Final reminder | N/A |
| Post-generation | Optional vibe coding rounds for images/enhancements |

---

## Validation Checklist

Before finalizing, verify:

- [ ] Categories are from the approved list
- [ ] ID is lowercase with hyphens (not underscores)
- [ ] ID matches markdown file name (without .md extension)
- [ ] ID matches the folder name containing the markdown file
- [ ] Language tag is populated with valid code
- [ ] Title starts with an action verb
- [ ] Overview section includes all required subsections
- [ ] Prerequisites includes Snowflake account signup link as first item
- [ ] Conclusion starts with "Congratulations! You've successfully..."
- [ ] No HTML is used in the markdown
- [ ] All code snippets are preserved exactly
- [ ] Headers do not exceed H4 (####)
- [ ] H2 headings are 3-4 words max
- [ ] All URL links in Related Resources are verified and working
- [ ] All URL links mentioned throughout the article are valid

---

## Conversion Rules

### Template Structure

Use this template for all conversions:

```
author: [Your Name] 
id: [unique-identifier-with-dash] 
categories: [comma-separated list where suitable category are taken from the categories list below but make sure the quickstart category is one of them.] 
language: en
summary: [One sentence describing what this guide covers] 
environments: web 
status: Published 
feedback link: https://github.com/Snowflake-Labs/sfguides/issues 


# [Article Title] 
<!-- ------------------------ --> 
## Overview 

[One to two paragraphs introducing the topic and what will be accomplished] 

### What You'll Learn 
- [Key learning objective 1] 
- [Key learning objective 2] 
- [Key learning objective 3] 
- [Add more as needed] 

### What You'll Build 
[Describe the end result/application/solution the reader will create] 

[Optional: Include screenshot or diagram of final result] 

### Prerequisites 
- Access to a [Snowflake account](https://signup.snowflake.com/?utm_source=snowflake-devrel&utm_medium=developer-guides&utm_cta=developer-guides)
- [Required software/tools] 
- [Required knowledge/skills] 
- [Other requirements] 

<!-- ------------------------ --> 
## Setup 

### [Setup Step 1 - e.g., Environment Configuration] 
[Detailed instructions] 

```[language] 
[code snippet if applicable] 
``` 

### [Setup Step 2 - e.g., Installation] 
[Detailed instructions] 

> IMPORTANT: 
> - [Critical note 1] 
> - [Critical note 2] 

<!-- ------------------------ --> 
## [Main Content Section 1] 

### [Subsection 1.1] 
[Detailed explanation] 

```[language] 
[code snippet if applicable] 
``` 

### [Subsection 1.2] 
[Detailed explanation] 

[Include screenshots/diagrams where helpful] 


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Snowflake-Labs/snowflake-demo-notebooks](https://github.com/Snowflake-Labs/snowflake-demo-notebooks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
