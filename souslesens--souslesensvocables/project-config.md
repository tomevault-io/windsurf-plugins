---
trigger: always_on
description: Shared cross-agent rules live in:
---

# Central Agent Rules

Shared cross-agent rules live in:

`C:\Users\kounnoughi\OneDrive - Jems\Documents\Obsidian Vault\AI\second-brain\Agent Rules\GLOBAL_AGENT_RULES.md`

Read that file for global behavior across Claude Code, Codex, and future agents. If Karim asks to change a global agent rule, update the central file first, then update relevant adapters according to:

`C:\Users\kounnoughi\OneDrive - Jems\Documents\Obsidian Vault\AI\second-brain\Agent Rules\RULE_CHANGE_PROTOCOL.md`

Local/project-specific rules in this file still apply after these shared rules.

---

# SousLeSens Vocables - Claude Code Guidelines


## Workspace Trust

Repo + worktrees = trusted local env.

Claude no ask confirmation for:
- Editing files
- Running git commands
- Applying multiple changes
in workspace.


# Claude Permissions

Claude allowed:
- Read, create, update, delete files in repo + worktrees
- Local git: add, commit, branch, diff
- Modify code freely, no per-change confirmation

Claude NOT allowed:
- Push commits to remote
- Create/modify GitHub PRs
- Modify GitHub issues/comments
- Any GitHub write action

Claude MUST:
- Ask explicit permission before GitHub write op
- Ask before push, even if told "finish task"

# Avoid useless comments

Write self‑explanatory code: expressive precise names + clear structure over comments.
Use descriptive identifiers (variables, functions, classes) conveying intent + domain meaning.
Comments only for:

Complex logic not obvious from code,
Business decisions / domain rationale,
Temporary patches/workarounds (include why + removal conditions),
Separating long multi‑step processes (section headers / brief overviews).


No redundant/obvious comments (e.g. "increment i", restating code).
Minimal comments via self‑documenting code: naming, structure, small focused functions

## Overview

Docs to help Claude Code work with SousLeSens Vocables codebase.

## Project Description

**SousLeSens Vocables** = semantic web platform for knowledge graph viz, ontology mgmt, SPARQL query building. Web UI for exploring + manipulating RDF/OWL ontologies in triple stores.

## Quick Reference

### Key Documentation Files

- **[claude.md](./claude.md)** (this file) - Main overview + quick reference

- **[refactoring-guidelines.md](./refactoring-guidelines.md)** - Code style + refactoring rules
- **[architecture.md](./architecture.md)** - System architecture deep dive
- **[module-patterns.md](./module-patterns.md)** - Common module patterns + examples
- **[sparql-guidelines.md](./sparql-guidelines.md)** - SPARQL execution + query building
- **[coding-standards.md](./coding-standards.md)** - Detailed coding standards

### Technology Stack

**Frontend:**
- JavaScript ES6+ module system (`import`/`export`)
- jQuery for DOM + AJAX
- Vis.js for graph viz
- JSTree for hierarchical trees
- Async.js for callback async flow

**Backend:**
- Node.js + Express.js
- SPARQL for semantic queries
- Triple store integration (Virtuoso, GraphDB, etc.)

**Build Tools:**
- Webpack for bundling
- ESLint for code quality

## Core Architecture Patterns

### 1. IIFE Module Pattern (Universal)

**Every module follows this pattern:**

```javascript
var ModuleName = (function () {
    var self = {};

    // Private variables (closure scope)
    var privateVar = null;

    // Public methods and properties
    self.publicMethod = function() {
        // Implementation
    };

    self.publicProperty = null;

    return self;
})();

// ES6 Module Export
export default ModuleName;

// Global Window Assignment (for backward compatibility with inline HTML)
window.ModuleName = ModuleName;
```

**Why both exports?**
- `export default` for ES6 module imports
- `window.ModuleName` for inline HTML handlers like `onclick="ModuleName.method()"`

### 2. Async Flow Control with async.js

**Codebase uses async.js (NOT async/await):**

```javascript
// Sequential execution
async.series([
    function(callbackSeries) {
        // Step 1
        callbackSeries(null, result);  // Or callbackSeries(err)
    },
    function(callbackSeries) {
        // Step 2 (waits for step 1)
        callbackSeries();
    }
], function(err, results) {
    if (err) {
        return callback(err);
    }
    // All steps complete
});

// Iterating array sequentially
async.eachSeries(items, function(item, callbackEach) {
    // Process item
    callbackEach();
}, function(err) {
    // All items processed
});

// Looping with condition
async.whilst(
    function() { return condition; },  // Test function
    function(callbackWhilst) {
        // Loop body
        callbackWhilst();
    },
    function(err) {
        // Loop complete
    }
);
```

### 3. Error-First Callbacks

**Node.js convention used throughout:**

```javascript
function doSomething(param, callback) {
    if (error) {
        return callback(err);  // First param is error
    }
    callback(null, result);  // null means no error
}

// Usage:
doSomething(param, function(err, result) {
    if (err) {
        return MainController.errorAlert(err);
    }
    // Use result
});
```

## Critical SPARQL Execution Pattern

### Main Method: Sparql_proxy.querySPARQL_GET_proxy

**Primary way to execute SPARQL queries:**

```javascript
var url = Config.sources[source].sparql_server.url + "?format=json&query=";

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [souslesens/souslesensVocables](https://github.com/souslesens/souslesensVocables) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
