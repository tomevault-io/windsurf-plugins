---
trigger: always_on
description: description: Generate hierarchical Inf diagram notes as YAML files in the inf-notes directory. First run creates root.yaml + Level 1. Subsequent runs scan all nodes across all levels and extend expandable ones by one level. Use --focus <file> to expand only a specific file.
---

---
name: inf
description: Generate hierarchical Inf diagram notes as YAML files in the inf-notes directory. First run creates root.yaml + Level 1. Subsequent runs scan all nodes across all levels and extend expandable ones by one level. Use --focus <file> to expand only a specific file.
---

# Inf Repository Notes Generator

Generate comprehensive visual documentation for the current repository using the Inf YAML format.

## Arguments

**--focus <file>** - Expand only a specific subgraph file and its children

```bash
/inf --focus api.yaml              # Only expand api.yaml
/inf --focus api__auth.yaml        # Only expand api__auth.yaml (nested file)
/inf --focus module-frontend.yaml  # Only expand module-frontend.yaml
```

**When to use `--focus`:**
- You want to deeply explore one specific area without touching others
- You're iterating on a particular module's documentation
- You need to regenerate/fix one subgraph without affecting the rest

**Behavior:**
- Scans only the specified file for nodes to expand
- Creates subgraph files only for that focused file's children (one level deep)
- Ignores all other YAML files in inf-notes/
- **One-level-at-a-time**: Allows controlled iteration and review before going deeper
- **Contrast**: Without `--focus`, scans ALL files at ALL levels for expandable nodes

**--yaml-convert** - Convert all YAML files to Inf JSON format

```bash
/inf --yaml-convert               # Convert inf-notes/*.yaml to JSON
```

**What it does:**
- Runs `python3 ~/.claude/skills/inf/scripts/yaml_convert.py --dir inf-notes/`
- Converts all YAML files in `inf-notes/` directory to Inf JSON format
- Computes layout positions using Graphviz
- Outputs JSON files alongside the YAML files

**When to use `--yaml-convert`:**
- After completing YAML creation and validation
- When you want to view diagrams in the Inf canvas application
- As a final step to generate viewable output

---

# YAML Format Specification

## Core Principle

- **Every YAML file MUST have a title and introduction:**
  - `title` node — The name or title of this graph (e.g., "Authentication Module")
  - `intro` node — A brief explanation of what this graph covers
- Provide a comprehensive overview (the full picture) at the root level (root.yaml).
- Place detailed explanations in separate YAML files, using file-based subgraphs.
- Use appropriate node types:
  - rectangle — concepts, components, modules
  - circle — entry / exit points, external systems
  - diamond — decisions, conditionals
  - text — details / annotations (no border)
  - code — commands, pseudocode, or source code snippets
  - table — data or comparisons (cells may contain subgraphs)
  - url — references / resources (use text/rectangle type with URL in content)
- Create meaningful connections:
    - Directed edges for flow or dependencies
    - Undirected edges for associations
- Use groups to organize related nodes, with clear visual boundaries and labels.
- Go as deep as needed — subgraphs support infinite nesting levels.
- Separate YAML files for each subgraph, with clear and descriptive names (e.g., module-authentication.yaml, concept-event-loop.yaml). Use relative paths only.
- It's fine to use a large text node or a code node to include details or pseudocode.
- **IMPORTANT**: Keep related content together in the same node. Use multiline text (with `\n`) to combine titles and content instead of creating separate nodes. The layout algorithm may place separate nodes far apart, breaking visual relationships. Example: Use `"Title\nContent here..."` instead of two separate nodes.
- **AVOID isolated nodes**: Don't create standalone title nodes or short text nodes without connections. Instead, use groups with descriptive names to organize related nodes. Groups provide better visual structure than isolated title nodes.


---

## Format

### Normal Node

```yaml
nodes:
  - text: "Node Name"
    type: [rectangle|circle|diamond|text|code|table]  # default: rectangle
    align: [left|center|right]                        # default: center
    attr: [title|intro]                               # optional: layout attribute
```

**Node Types:**
- `rectangle` - Concepts, process steps, components, modules (default)
- `circle` - Start/end points, states, actors, external systems
- `diamond` - Decision points, conditionals, gateways
- `text` - Titles, labels, details, annotations (no border)
- `code` - Code snippets, configuration, commands, scripts

**URLs/References:**
URLs are automatically detected and highlighted in any node type (clickable with Ctrl+Click):
```yaml
- text: "API Docs\nhttps://api.example.com/docs"
  type: text
  align: left
```

**Text Alignment:**
- `left` - For lists, code blocks
- `center` - For titles, labels (default)
- `right` - For dates, metadata

**Layout Attributes (`attr`) - REQUIRED:**

Every YAML file must start with a `title` and `intro` node:

```yaml
nodes:
  # REQUIRED: Title - the name of this graph

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jysh1214/inf](https://github.com/jysh1214/inf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
