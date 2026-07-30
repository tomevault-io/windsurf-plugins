---
trigger: always_on
description: **IMPORTANT**: This configuration applies to task routing and agent empowerment.
---

# Claude Code CLI - Agent Execution Framework

## 🎯 AGENT EXECUTION CONTEXT

**IMPORTANT**: This configuration applies to task routing and agent empowerment.

**Specialized agents have FULL TOOL ACCESS and execute tasks directly within their domains of expertise.**

---

## 🤖 AVAILABLE IMPLEMENTATION AGENTS

| Agent | Implementation Domain |
|-------|---------------------|
| **comprehensive-analyst** | Deep analysis, evaluation, and investigation |
| **code-review-gatekeeper** | Code review, quality validation, testing |
| **peer-review-critic** | **Final gatekeeper** — independent, diff-scoped critical peer review of branch-vs-base before work is declared done (runs after code-review-gatekeeper) |
| **spec-compliance-reviewer** | Requirement-by-requirement spec conformance review — verifies the build against `specs/<name>.md` in the spec → build → review loop |
| **devops-orchestrator** | Infrastructure, CI/CD, deployment automation |
| **rust-expert** | Rust systems programming, high-performance applications, CLI tools |
| **csharp-expert** | C#/.NET development, ASP.NET Core, Azure solutions |
| **go-expert** | Go development, microservices, cloud-native applications |
| **java-expert** | Java/Spring Boot development, enterprise applications, Android |
| **python-expert** | Python development, web frameworks, data science, automation |
| **typescript-expert** | TypeScript/JavaScript development, React/Next.js, Node.js backends |
| **mql-trading-dev** | MQL4/MQL5 and C/C++ development for MetaTrader, Expert Advisors, indicators, trading systems |
| **powershell-expert** | Windows command-line executor — runs delegated shell work; PowerShell automation, Windows administration |
| **bash-expert** | Command-line executor — runs delegated shell work; Bash/POSIX scripting, Linux/CI automation |
| **database-specialist** | Database design, schema optimization, query optimization, SQL/NoSQL |
| **frontend-specialist** | Frontend UI development, React/Vue/Angular, responsive design |
| **security-specialist** | Security audits, vulnerability assessment, authentication, compliance |
| **system-architect** | System architecture design, technical decisions, scalability patterns |
| **technical-docs-writer** | Documentation, guides, API documentation, developer guides |
| **uiux-specialist** | UI/UX design, accessibility, design systems, user flows |
| **product-owner** | Requirements, user stories, project planning, backlog management |

---

## 🎯 TASK ROUTING GUIDELINES

### Language-Specific Development
- **Rust development** → rust-expert
- **C#/.NET development** → csharp-expert
- **Go development** → go-expert
- **Java/Spring Boot** → java-expert
- **Python development** → python-expert
- **TypeScript/JavaScript** → typescript-expert
- **MQL4/MQL5 & MetaTrader trading systems** → mql-trading-dev

### Scripting & Automation
- **Any shell command a task needs run** → bash-expert (POSIX/CI) or powershell-expert (Windows) — see the blanket execution policy below
- **Bash/shell script authoring** → bash-expert
- **PowerShell automation** → powershell-expert
- **Infrastructure automation** → devops-orchestrator

### Specialized Domains
- **Database design/optimization** → database-specialist
- **Frontend UI development** → frontend-specialist
- **UI/UX design** → uiux-specialist
- **Security audits** → security-specialist
- **Infrastructure/CI/CD** → devops-orchestrator
- **System architecture** → system-architect

### Analysis & Quality
- **Deep analysis/investigation** → comprehensive-analyst
- **Code review/validation** → code-review-gatekeeper
- **Final independent peer review (branch vs base, before "done")** → peer-review-critic
- **Spec conformance review (build vs `specs/<name>.md`)** → spec-compliance-reviewer

### Documentation & Planning
- **Technical documentation** → technical-docs-writer
- **Requirements/user stories** → product-owner

### Agent Capabilities
- Agents have FULL TOOL ACCESS within their domain of expertise
- Agents read and write files directly without requesting permission
- Agents execute commands and run tests as needed, within the command-line execution policy below
- Agents create implementations, configurations, and documentation
- Agents validate and test their work independently
- Agents make technical decisions within their specialization
- Agents can invoke other agents when cross-domain expertise is needed

---

## 🚀 EXECUTION PRINCIPLES

### Agent Empowerment
- Agents have unrestricted access to tools within their domain
- Agents implement solutions directly; the one routine onward delegation is command-line work, per the policy below
- Agents create concrete deliverables and working implementations
- Agents validate their work and ensure quality standards
- Agents operate autonomously with full technical authority

### 🖥️ Command-line Execution Policy (blanket)

**Principle**: Delegate when the output is large and the conclusion is small.

- Delegate **all shell execution** — builds, tests, git and gh operations, JSON/YAML
  processing with jq/yq, log grinding, any shell command — to the executor agents:
  **bash-expert** (POSIX shell, Git Bash, Linux/CI/containers) or **powershell-expert**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tomas-rampas/claude-agentic-framework](https://github.com/tomas-rampas/claude-agentic-framework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
