---
trigger: always_on
description: This document provides guidance for LLM agents working on the Polydoc project, a Clojure-based tool for building documentation systems using Pandoc filters.
---

# Agents Guide for Polydoc Development

This document provides guidance for LLM agents working on the Polydoc project, a Clojure-based tool for building documentation systems using Pandoc filters.

## Project Overview

**Polydoc** is a documentation processing tool that brings Pandoc's powerful filtering capabilities to the JVM/Clojure ecosystem. It aims to provide:

- Advanced Pandoc filters compiled with GraalVM for reduced latency
- JVM-native documentation tooling (no Python/Node.js dependencies)
- Support for multiple document formats via Pandoc
- Interactive documentation viewing with SQLite-powered search
- LLM-driven document processing capabilities

### Key Capabilities

The project implements several types of Pandoc filters:

- **Code execution filters**: Run Clojure, SQLite, JavaScript, Python code blocks
- **Rendering filters**: Process PlantUML diagrams
- **Linting filters**: Check Clojure code with clj-kondo
- **Include filters**: Compose documents from multiple sources
- **Build system**: Generate books with table of contents and full-text search
- **Interactive viewer**: HTTP-based document browser with search

## Allowed Command-Line Tools

As an LLM agent, you are restricted to using **only** these command-line tools:

1. **clj-nrepl-eval** - For evaluating Clojure code via nREPL
2. **clojure-skills** - For searching skills, managing plans, and tracking tasks
3. **clj-paren-repair** - For automatically fixing delimiter errors in Clojure files

All other interactions must happen through the **REPL** using the connected nREPL server (port 7889).

### clj-paren-repair Tool

The `clj-paren-repair` tool automatically detects and fixes delimiter (parentheses, brackets, braces) errors in Clojure files. It's particularly useful after making edits that may have introduced unbalanced delimiters.

**Usage:**
```bash
# Fix delimiter errors in a single file
clj-paren-repair path/to/file.clj

# Fix multiple files at once
clj-paren-repair file1.clj file2.clj file3.clj

# Fix all Clojure files in a directory
clj-paren-repair src/**/*.clj
```

**When to use:**
- After making edits that result in "EOF while reading" errors
- When encountering "Unmatched delimiter" compiler errors
- As a quick fix for parenthesis-related syntax errors
- Before committing changes to ensure proper formatting

**Example:**
```bash
$ clj-paren-repair src/polydoc/viewer/server.clj
Reformatted src/polydoc/viewer/server.clj

clj-paren-repair Results
========================

  src/polydoc/viewer/server.clj: Delimiter errors fixed and formatted [delimiter-fixed, formatted]

Summary:
  Success: 1
  Failed:  0
```

**Note:** The tool uses parinfer-rust (if available) or parinferish as a fallback to intelligently repair delimiters based on indentation and context.

## Fish Shell Usage

This project uses the **fish shell**. Fish has different syntax than bash/zsh, especially for multi-line strings and heredocs.

### ❌ WRONG (bash heredoc syntax doesn't work in fish):

```bash
cat > file.txt << 'EOF'
Line 1
Line 2
EOF
```

### ✅ CORRECT Fish shell patterns:

**1. Multi-line strings in echo:**
```fish
echo "Line 1
Line 2
Line 3"
```

**2. Use printf for complex output:**
```fish
printf '%s\n' "Line 1" "Line 2" "Line 3" > file.txt
```

**3. Use command substitution with echo:**
```fish
echo (echo "Line 1"; echo "Line 2"; echo "Line 3") > file.txt
```

**4. For creating files, use the write tool instead:**
- Prefer the `write` tool for creating files with content
- This avoids shell syntax issues entirely

**5. For clj-nrepl-eval with multi-line Clojure code:**
```fish
# WRONG - heredoc doesn't work
clj-nrepl-eval -p 7889 <<'EOF'
(defn foo [x] x)
EOF

# CORRECT - use double quotes and escape as needed
clj-nrepl-eval -p 7889 "(defn foo [x] (println x) x)"

# CORRECT - for complex code, write to file first, then eval
# Or use parentheses to wrap multi-line Clojure
```

**Key differences from bash:**
- No `<<EOF` heredoc syntax
- Multi-line strings work directly in double quotes
- Newlines in quoted strings are preserved
- Use `printf` for formatted output
- Command substitution uses `()` not `$()`

**Best practice:** For any complex file content creation, use the `write` tool instead of shell commands.

## Dependencies Analysis

### Core Dependencies (deps.edn)

```clojure
{:deps
 {org.clojure/clojure {:mvn/version "1.12.3"}           ; Latest Clojure
  
  ;; CLI & Terminal
  cli-matic/cli-matic {:mvn/version "0.5.4"}           ; CLI interface
  io.github.paintparty/bling {:mvn/version "0.8.8"}    ; Terminal formatting
  
  ;; Data Processing
  org.clojure/data.json {:mvn/version "2.5.1"}         ; JSON handling (Pandoc AST)
  clj-commons/clj-yaml {:mvn/version "1.0.29"}         ; YAML support
  org.clj-commons/pretty {:mvn/version "3.6.7"}        ; Pretty printing
  mvxcvi/puget {:mvn/version "1.3.4"}                  ; Pretty printing
  
  ;; Data Structures & Validation
  metosin/malli {:mvn/version "0.19.2"}                ; Schema validation
  funcool/lentes {:mvn/version "1.3.3"}                ; Functional lenses
  
  ;; Database
  com.github.seancorfield/honeysql {:mvn/version "2.7.1350"}   ; SQL DSL

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iwillig/llm-planner](https://github.com/iwillig/llm-planner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
