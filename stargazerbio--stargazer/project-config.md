---
trigger: always_on
description: **General Guidelines**
---

# Stargazer

**General Guidelines**
- Always call `ToolSearch` to fetch the schema of any deferred MCP tool before invoking it.
- When I say task, I am referring to a Flyte V2 task, not a raw python function
- Tasks are collected into workflows which are just regular tasks calling other tasks, sync or async
- This project uses UV so the appropriate commands are `uv add` and `uv pip install -e .`
- If something is changed that you didn't change, it's not a typo, it's a manual change. I do still write code occassionally..
- Don't use the "if TYPE_CHECKING:" pattern anywhere, Flyte will always check types
- Do not make any git commits unless explicitly requested
- The README is a document written exclusively BY HUMANS FOR HUMANS. Never modify the README. Notify if it is out of spec only.

**Dev Process**
- You will implement features piece by piece in a sequential fashion
- Handle a single case well at first instead of trying to anticipate every way the app will be used
- Do not add complexity until it is needed, which may be never
- Simple tests will be written before implementation and you will pause to ensure they're capturing the right behavior
- Implementation will be tightly scoped so it can be understood
- Tests will run until they pass
- All necessary CLI tools e.g. parabricks, bwa etc, are available in PATH. Use them to generate test assets as needed and alert the user if they are not available.
- When adding a task that wraps a new CLI tool, check the `Dockerfile` bioconda install block to confirm the tool is listed. If it is missing, add it and notify the user.
- **CRITICAL** Do not consider backwards compatibility unless explicitly requested!
- Run `ruff --fix` after every set of changes to satisfy the pre-commit

## OpenCode Agent Definitions

The `.opencode/agent/` directory contains specialized agent definitions for [OpenCode](https://github.com/sst/opencode), an AI coding assistant. These markdown files define role-specific personas that can be invoked as subagents, each with tailored instructions, temperature settings, and tool permissions.

### Available Agents

| Agent | File | Purpose |
|-------|------|---------|
| **Architecture** | `architecture.md` | Designs feature plans in `.opencode/plans/` and maintains docs in `docs/` |
| **Task** | `task.md` | Implements individual Flyte v2 tasks for bioinformatics tools |
| **Test** | `test.md` | Writes unit and integration tests following TDD approach |
| **Workflow** | `workflow.md` | Composes Flyte v2 tasks into end-to-end pipelines |
| **Code Review** | `code-review.md` | Strict code reviewer that audits for edge cases, UX issues, and data provenance |

### Agent File Format

Each agent file uses YAML frontmatter to configure behavior:
```yaml
---
description: Brief description of the agent's role
mode: subagent
temperature: 0.2  # Lower = more deterministic
tools:
  write: true
  edit: true
  bash: true
---
```

The markdown body contains detailed instructions including:
- Role definition and core principles
- Implementation templates and patterns
- Project-specific rules (imports, async patterns, types)
- Checklists and communication guidelines

### When to Use

- **architecture agent**: When designing a new feature or updating system specs
- **task agent**: When implementing a new bioinformatics tool wrapper
- **test agent**: When writing tests for tasks or workflows
- **workflow agent**: When composing tasks into pipelines
- **code-review agent**: Before merging code, to catch issues early

## Docstring Spec References

Every module in `src/` has two conventions in its module-level docstring:

1. The first line is a `###` heading so it renders prominently in the generated API docs.
2. A `spec:` line at the bottom is a markdown link to the relevant architecture doc:

```
spec: [docs/architecture/types.md](../architecture/types.md)
```

**Rationale:** This serves two purposes:
1. **Diff scanning** — when reviewing recent PRs or commits, an LLM can immediately see which spec doc is affected by any changed module and check whether the docs need updating.
2. **Low-overhead lookup** — when making changes to a specific module, the relevant high-level architecture is one link away without any search.

The `spec:` line is **module-level only** — class and function docstrings do not carry it.

100% docstring coverage is enforced by the `docstr-coverage` pre-commit hook.

## Specs, Plans and Reference Materials

- **`.opencode/reference/flyte_v2_docs.md`** - Official Flyte v2 documentation
- **`.opencode/reference/sdk_examples_concise.md`** - Flyte SDK v2 examples
- **`.opencode/reference/tool_refs/`** - Bioinformatics tool documentation, use as the source of truth for tool parameters and behavior
- **`docs/`** - Project documentation (architecture, guides, reference)
  - **Critical**: Docs must be updated as the project evolves to stay in sync with the current state
  - No code in architecture docs - these are high-level references supported by docstrings in the actual functions
  - Guides are the only docs that contain code examples
- **`.opencode/plans/`** - Step by step instructions for building new features and fixing bugs
  - Only place outside src where code snippets are allowed
  - Keep track of progress and check off completed work as you go

## Project Structure


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [StargazerBio/stargazer](https://github.com/StargazerBio/stargazer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
