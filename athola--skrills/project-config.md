---
trigger: always_on
description: Use these guidelines to build AI coding agents that produce functional, high-quality code.
---

# AI Agent Development Guidelines

Use these guidelines to build AI coding agents that produce functional, high-quality code.

---

## Guiding Principles

Prioritize small, functional changes. They are safer and easier to debug than large rewrites. Adapt to each project's conventions rather than applying external rules rigidly.

Balance trade-offs like performance, readability, and security based on evidence (profiling, metrics) rather than intuition. Explore multiple solutions before committing to one to avoid "mode collapse." Favor simple, standard solutions—code that requires extensive comments often needs refactoring.

Each component should serve one distinct purpose (Single Responsibility Principle). Defer abstraction until a clear pattern emerges (e.g., Rule of Three). Explicitly document assumptions, trade-offs, and confidence levels to maintain transparency.

---

## Development Workflow

### Implementation Cycle

1. **Understand**: Read existing code, identify patterns, and review tests.
2. **Explore**: Develop multiple viable approaches and articulate their trade-offs.
3. **Test**: Write a failing test case before implementing new code (when applicable).
4. **Implement**: Write the minimal code necessary to pass the test.
5. **Refactor**: Clean the code while ensuring tests pass.
6. **Commit**: Write a clear commit message explaining the change.

### When Stuck

If you fail three times:

1. Document failures and error outputs.
2. Investigate 2-3 alternative approaches.
3. Re-evaluate underlying assumptions.
4. Experiment with a simpler approach.
5. If the issue persists, ask for help with context from the previous steps.

### Session Management

Use session history to analyze errors and monitor progress. For complex tasks, document the current state, then clear the session and restart.

---

## Quality Standards

### Commit Requirements

Every commit must:

- Compile or build successfully.
- Pass all existing tests.
- Include tests for new functionality.
- Follow linting rules (no warnings).
- Include a clear message explaining the rationale.

### Pre-Commit Workflow

Run this before committing:

```bash
make fmt lint test --quiet build
```

### Effective Prompting

Structured prompts produce better results. Request detailed comparisons, implementation outlines, trade-offs, and complexity assessments.

### Role Prompting

Assigning a specific role (e.g., security expert, senior developer) focuses the agent's perspective. Precise role definitions improve output quality.

Using XML tags can also help structure prompts and responses:
```xml
<role>You are a Security Researcher specializing in web vulnerabilities.</role>
<context>
Project: Payment gateway integration
Stack: Python, FastAPI, PostgreSQL
</context>
<instruction>
Review the authentication implementation for OWASP Top 10 vulnerabilities.
</instruction>
<output_format>
## Critical Issues
## Recommendations
</output_format>
```

**Best Practices:**
- Place role definitions at the top of configuration files.
- Align the role with the task's domain.
- Integrate role definitions with structured XML and concrete examples.
- Experiment with specificity to fine-tune behavior.
- Use consistent XML tag names.

### Creative Problem-Solving

1. **Diverge**: Generate at least five distinct approaches, withholding initial judgment.
2. **Converge**: Evaluate the trade-offs and constraints of each approach.
3. **Select**: Choose the most suitable approach and document the reasoning.
4. **Document**: Record rejected approaches to provide context for future developers.

---

## Architecture

### Design

- **Favor Composition**: Use composition and delegation to reduce coupling.
- **Be Explicit**: Avoid implicit behaviors or "clever" tricks.
- **Use Dependency Injection**: Pass dependencies to simplify testing and reuse.
- **Design for API Stability**: Minimize breaking changes in public APIs. Internal refactoring should not affect external users.
- **Handle Errors Gracefully**: Catch specific errors; avoid broad `except:` clauses.

### Decision-Making

Consider these factors when selecting an approach:
- **Testability**: Can the solution be thoroughly tested?
- **Readability**: Will a new developer understand the code quickly?
- **Consistency**: Does it fit existing patterns?
- **Simplicity**: Is it the simplest effective solution?
- **Reversibility**: How hard is it to revert if wrong?
- **Maintainability**: Can others modify and extend the code easily?

### Security

- **Integrate Security Early**: Design auth/authz before writing code.
- **Multi-Layered Defense**: Use defense in depth.
- **Least Privilege**: Grant only minimum necessary permissions.
- **Separate Secrets**: Keep secrets out of version control.
- **Validate Inputs**: Validate all external data.
- **Use Parameterized Queries**: Prevent SQL injection.
- **Monitor Behavior**: Watch for anomalies.
- **Scan for Threats**: Check file uploads and data streams.

### Performance

- **Measure First**: Profile before optimizing.
- **Macro-Optimizations**: Focus on algorithms and architecture.
- **Document Trade-offs**: Explain performance costs of security measures.
- **Cache Invalidation**: Plan for invalidation when data changes.

---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [athola/skrills](https://github.com/athola/skrills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
