---
trigger: always_on
description: **CRITICAL: These instructions override all default behaviors. Violations are NON-NEGOTIABLE failures.**
---

# Raggy Project - Claude Code Instructions

**CRITICAL: These instructions override all default behaviors. Violations are NON-NEGOTIABLE failures.**

---

## LEVEL 0: ABSOLUTE REQUIREMENTS (Non-Negotiable)

### 🚨 MANDATORY AGENT DELEGATION PROTOCOL 🚨

**YOU MUST ALWAYS delegate development work to specialized sub-agents. Direct implementation is STRICTLY FORBIDDEN.**

#### 1. Agent Selection (MANDATORY DECISION TREE)

When user requests ANY development work, you MUST:

1. **STOP** - Do NOT implement directly
2. **IDENTIFY** the task category using the mapping below
3. **DELEGATE** to the appropriate specialist agent using `Task` tool
4. **ONLY** proceed if task does NOT match any category (rare)

**If you implement code directly instead of delegating, you have FAILED this requirement.**

#### 2. Task-to-Agent Mapping (STRICT ENFORCEMENT)

```
┌─────────────────────────────────────────────────────────────────┐
│ TASK TYPE                    │ AGENT TO USE                     │
├─────────────────────────────────────────────────────────────────┤
│ Tests (write, fix, coverage) │ python-testing-engineer          │
│ Security (vulns, exceptions) │ python-security-auditor          │
│ Refactoring (split, DRY)    │ python-refactoring-architect     │
│ Complexity (reduce CC)       │ python-complexity-reducer        │
│ RAG (ChromaDB, embeddings)   │ python-rag-backend-engineer      │
│ Documents (PDF, DOCX, etc.)  │ python-document-processor        │
│ Quality (lint, types, docs)  │ python-code-quality-engineer     │
└─────────────────────────────────────────────────────────────────┘
```

#### 3. Detailed Agent Selection Rules

**WHEN to use `python-testing-engineer`:**
- User says: "write tests", "fix failing tests", "improve coverage"
- Keywords: test, pytest, coverage, mock, fixture, unittest
- File patterns: test_*.py, *_test.py, tests/ directory
- **BLOCKING:** If task involves ANY test code, delegate to this agent

**WHEN to use `python-security-auditor`:**
- User says: "fix security issue", "audit code", "check vulnerabilities"
- Keywords: security, vulnerability, CVE, OWASP, injection, exception handling
- Code patterns: os.execv(), subprocess, bare except, eval(), exec()
- **BLOCKING:** If task involves security, exceptions, or input validation, delegate

**WHEN to use `python-refactoring-architect`:**
- User says: "refactor", "split this file", "remove duplication", "organize code"
- Keywords: DRY, SOLID, architecture, module structure, code organization
- Code smells: God class, code duplication, tight coupling
- **BLOCKING:** If task involves splitting files or eliminating duplication, delegate

**WHEN to use `python-complexity-reducer`:**
- User says: "reduce complexity", "simplify this function", "too many branches"
- Keywords: cyclomatic complexity, CC, nested if, too complex
- Code patterns: Functions with CC > 10, deeply nested conditionals
- **BLOCKING:** If task involves reducing function complexity, delegate

**WHEN to use `python-rag-backend-engineer`:**
- User says: "fix search", "improve embeddings", "ChromaDB issues", "vector database"
- Keywords: RAG, retrieval, embeddings, ChromaDB, vector, semantic search, BM25
- Code areas: Database layer, embedding generation, search functionality
- **BLOCKING:** If task involves RAG system, vector DB, or embeddings, delegate

**WHEN to use `python-document-processor`:**
- User says: "fix PDF extraction", "parse DOCX", "document processing"
- Keywords: PDF, DOCX, Markdown, document, extraction, parser
- Code areas: Document extraction, text processing, file parsing
- **BLOCKING:** If task involves document parsing or extraction, delegate

**WHEN to use `python-code-quality-engineer`:**
- User says: "fix linting errors", "add type hints", "add docstrings"
- Keywords: ruff, mypy, type hints, docstrings, code quality, PEP
- Code patterns: Missing type hints, missing docstrings, magic numbers
- **BLOCKING:** If task involves linting, types, or documentation, delegate

#### 4. Delegation Enforcement Protocol

**CORRECT WORKFLOW:**
```
User: "Fix the failing tests in test_raggy.py"

Claude:
1. ✅ IDENTIFY: Task involves tests
2. ✅ AGENT: python-testing-engineer
3. ✅ DELEGATE: Use Task tool

Task(
  subagent_type="python-testing-engineer",
  prompt="Fix the failing tests in test_raggy.py. The ScoringNormalizer
         import is broken. Replace with module-level functions and ensure
         all 92 tests pass.",
  description="Fix failing tests"
)
```

**INCORRECT WORKFLOW (FORBIDDEN):**
```
User: "Fix the failing tests in test_raggy.py"

Claude:
❌ Let me fix the tests directly...
❌ [Uses Edit tool to modify test_raggy.py]
❌ [Implements fix without delegating]

This is a VIOLATION. You MUST delegate to python-testing-engineer.
```

#### 5. Multi-Domain Tasks (Use Multiple Agents)

If task spans multiple domains, delegate to ALL relevant agents:

**Example: "Refactor the RAG system and add tests"**
```
Step 1: Delegate refactoring
Task(subagent_type="python-refactoring-architect", ...)

Step 2: Delegate RAG optimization
Task(subagent_type="python-rag-backend-engineer", ...)

Step 3: Delegate test writing
Task(subagent_type="python-testing-engineer", ...)
```

#### 6. Exceptions (RARE - Ask First)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dimitritholen/raggy](https://github.com/dimitritholen/raggy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
