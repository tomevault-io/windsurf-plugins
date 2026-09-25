---
trigger: always_on
description: This document provides guidance for AI agents, large language models (LLMs), and AI-assisted development tools interacting with the Chia repository. These guidelines ensure that AI-generated contributions meet the project's quality, security, and compliance standards.
---

# AI Agent Guidelines

This document provides guidance for AI agents, large language models (LLMs), and AI-assisted development tools interacting with the Chia repository. These guidelines ensure that AI-generated contributions meet the project's quality, security, and compliance standards.

## Project Context

### Overview

CHIA is an open-source framework for agile and principled hardware/software co-design research. Even though many of the steps of the hardware/software co-design process can be accelerated by AI, existing research using AI in these contexts has been limited to small studies on isolated examples because it is still too hard to assemble more complex experiments. CHIA solves this problem by enabling users to express the whole co-design **workflow** in an agile way with all of the tools you already use. CHIA abstracts workflows as graphs, and provides an efficient, feature rich runtime system to execute these workflows.

Chia has extensive documentation located in the `docs/` folder and at https://docs.chialoops.ai/en/latest, which you should read to find how to work with the existing Chia codebase.

A published research paper describing Chia is available here: https://arxiv.org/abs/2606.27350, which provides case studies and examples of how Chia can be used.

### Architecture & Key Concepts

AI Agents should review these documentation to learn about the Architecutre & Key Concepts in Chia before making any changes to the repository:

- **[CHIA Basics](https://docs.chialoops.ai/en/latest/getting-started/chia-basics.html)** — the core ideas, start here.
- **[Architecture Overview](https://docs.chialoops.ai/en/latest/concepts/overview.html)** — how CHIA works under the hood.

User guides:

- [ChiaFunction](https://docs.chialoops.ai/en/latest/user_guides/chia_function.html)
- [ChiaTool](https://docs.chialoops.ai/en/latest/user_guides/chia_tool.html)
- [Cluster Configuration Reference](https://docs.chialoops.ai/en/latest/user_guides/cluster_config_reference.html)
- [Building a CHIA-compatible Docker image](https://docs.chialoops.ai/en/latest/user_guides/docker_images.html)
- [Caching and Bypass](https://docs.chialoops.ai/en/latest/user_guides/caching_and_bypass.html)
- [Profiling](https://docs.chialoops.ai/en/latest/user_guides/profiling.html)

## Coding Conventions

AI agents MUST follow existing coding conventions in the Chia codebase when making contributions & changes.

## Contribution Guidelines for AI Agents

### General Requirements

AI-generated contributions are welcome and MUST meet the gudielines outlined in [CONTRIBUTING.md](CONTRIBUTING.md)

1. **Disclosure**: Pull requests containing AI-generated code SHOULD disclose this in the PR description, including the AI tool used.

2. **Human Review**: All AI-generated contributions MUST be reviewed by a human maintainer before merging.

3. **Testing**: AI-generated code MUST include appropriate tests and pass all existing tests.

4. **License Compliance**: AI agents MUST NOT introduce code that violates the project's license or includes incompatibly licensed dependencies.

### Contribution Workflow

1. **Understand the task**: Review relevant issues, documentation, and existing code before generating changes.

2. **Make focused changes**: Keep contributions small and focused on a single issue or feature.

3. **Follow existing patterns**: Match the style and patterns of the surrounding code.

4. **Include tests**: Write tests for new functionality and ensure existing tests pass.

5. **Update documentation**: Update or create documentation as needed.

6. **Self-review**: Review generated code for errors, security issues, and adherence to guidelines before submission.

### Commit Messages

AI agents SHOULD follow this commit message format:

```
<type>(<scope>): <short summary>

<detailed description if needed>

Assisted-by: <AI Agent Name/Model>
Signed-off-by: Human Operator Name <email@example.com>
```

Types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`

## Prohibited Actions

AI agents MUST NOT:

- **Modify governance files** without explicit human instruction: STEERING_COMMITTEE.md, CODE_OF_CONDUCT.md, LICENSE, README.md, SECURITY.md, CONTRIBUTING.md
- **Bypass security controls**: Do not disable security checks, remove authentication, or weaken access controls
- **Introduce breaking changes** without explicit approval and documentation
- **Remove or weaken tests**: Do not delete tests or reduce test coverage without justification
- **Add dependencies** without security review and license verification
- **Access secrets or credentials**: Do not hardcode, log, or expose sensitive information
- **Ignore CI/CD failures**: Do not submit code that fails automated checks
- **Make changes outside the scope** of the assigned task
- **Generate code from memory** that may be copied from copyrighted sources without proper attribution

## Security Requirements

AI agents MUST follow these security practices:

- **Input validation**: Validate and sanitize all inputs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ucb-bar/chia](https://github.com/ucb-bar/chia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
