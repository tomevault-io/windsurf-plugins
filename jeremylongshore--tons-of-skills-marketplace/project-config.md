---
trigger: always_on
description: How agent definitions work in Claude Code plugins — creating specialized AI personas with capabilities, tool restrictions, and autonomous multi-step workflows using agents/*.md files.
---


Agents are specialized AI personas defined within Claude Code plugins. Where skills teach Claude *what* to do in a specific domain, agents define *who* Claude becomes when tackling a particular class of problem. An agent carries a distinct identity with its own capabilities, tool restrictions, and behavioral parameters -- making it suitable for complex, autonomous, multi-step workflows.

## When to Use Agents vs Skills

The distinction between agents and skills is fundamental to designing effective plugins. Choosing the wrong abstraction leads to either overly constrained workflows (using skills where agents are needed) or unnecessarily broad permissions (using agents where skills would suffice).

| Characteristic | Skills | Agents |
|---------------|--------|--------|
| **Activation** | Auto-activates based on context | Explicitly delegated or invoked |
| **Tool model** | Allowlist (`allowed-tools`) | Denylist (`disallowedTools`) |
| **Scope** | Focused, single-task | Broad, multi-step workflows |
| **Autonomy** | Follows instructions within one turn | Can iterate autonomously over multiple turns |
| **Identity** | Adds knowledge to Claude | Gives Claude a specialized persona |
| **Typical length** | 500-2,000 words | 200-1,000 words (directives, not procedures) |

**Use a skill when:**

- The task is focused and well-defined (e.g., "write tests for this component")
- You want auto-activation based on context matching
- You need precise tool restrictions (only allow specific tools)
- The instruction set is procedural: step 1, step 2, step 3

**Use an agent when:**

- The task requires autonomous exploration and decision-making
- Multiple tool categories are needed, with only a few excluded
- The agent needs to iterate (try, evaluate, adjust) over multiple turns
- You want Claude to adopt a specific professional persona (security auditor, UX researcher, etc.)
- The work involves judgment calls that vary based on what the agent discovers

## Agent File Structure

Agent definitions live in the `agents/` directory of a plugin. Each agent is a single markdown file with YAML frontmatter:

```
my-plugin/
└── agents/
    ├── security-auditor.md
    ├── performance-analyst.md
    └── code-archaeologist.md
```

### Complete Agent Example

```markdown
---
name: security-auditor
description: "Security-focused code reviewer specializing in OWASP Top 10, dependency vulnerabilities, and secrets detection"
capabilities:
  - "Static analysis of source code for security vulnerabilities"
  - "Dependency audit using npm audit and Snyk patterns"
  - "Secrets and credential detection in code and configuration"
  - "OWASP Top 10 compliance checking"
model: sonnet
effort: high
maxTurns: 15
disallowedTools:
  - "WebFetch"
  - "WebSearch"
expertise_level: expert
activation_priority: high
---

You are a senior application security engineer conducting a thorough security
review. Your primary objective is to identify vulnerabilities before code
reaches production.

## Review Methodology

1. **Reconnaissance**: Map the attack surface by identifying entry points
   (API routes, form handlers, file uploads, WebSocket endpoints).

2. **Dependency Analysis**: Check `package.json`, `requirements.txt`, or
   equivalent for known vulnerable dependencies. Flag any package with
   a critical or high severity CVE.

3. **Code Analysis**: Systematically review for:
   - SQL injection and NoSQL injection
   - Cross-site scripting (XSS) in rendered output
   - Insecure deserialization
   - Hardcoded secrets, API keys, or credentials
   - Missing authentication or authorization checks
   - Path traversal vulnerabilities
   - Server-side request forgery (SSRF)

4. **Configuration Review**: Check for insecure defaults in:
   - CORS policies
   - Cookie settings (HttpOnly, Secure, SameSite)
   - TLS configuration
   - Error handling (information leakage)

5. **Report**: Categorize findings by severity (Critical, High, Medium, Low)
   with specific file locations, code snippets, and remediation guidance.

## Output Format

Present findings as a structured security report:

- **Finding ID**: SEC-001, SEC-002, etc.
- **Severity**: Critical / High / Medium / Low
- **Category**: OWASP category (e.g., A03:2021 Injection)
- **Location**: File path and line number
- **Description**: What the vulnerability is
- **Impact**: What an attacker could do
- **Remediation**: Specific code changes to fix it
```

## Frontmatter Fields

### Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Unique agent identifier within the plugin (kebab-case) |
| `description` | string | 20-200 character summary of the agent's specialty |

### Optional Fields

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `capabilities` | array | [] | List of capabilities the agent provides |
| `model` | string | (default) | LLM model override: `sonnet`, `haiku`, or `opus` |
| `effort` | string | medium | Reasoning effort level: `low`, `medium`, or `high` |
| `maxTurns` | number | (default) | Maximum iterations in the agentic loop |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jeremylongshore/tons-of-skills-marketplace](https://github.com/jeremylongshore/tons-of-skills-marketplace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
