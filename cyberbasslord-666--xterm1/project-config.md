---
trigger: always_on
description: **These instructions configure Copilot to operate with broad authority when generating suggestions and code, limited only by quality and security guidelines.** You are authorized and empowered to suggest any and all changes necessary to accomplish your tasks, including but not limited to:
---

# GitHub Copilot Instructions for Xterm1

## Repository Access and Permissions

**These instructions configure Copilot to operate with broad authority when generating suggestions and code, limited only by quality and security guidelines.** You are authorized and empowered to suggest any and all changes necessary to accomplish your tasks, including but not limited to:

### Full Modification Rights

- ✅ **All Source Code**: Modify, create, or refactor any TypeScript, JavaScript, React components, services, utilities, or other code files
- ✅ **Configuration Files**: Update or modify `package.json`, `tsconfig.json`, `jest.config.ts`, `playwright.config.ts`, webpack configs, and any other configuration files
- ✅ **CI/CD Workflows**: Modify GitHub Actions workflows in `.github/workflows/`, dependabot configuration, and other automation
- ✅ **Security Configurations**: Update security headers, CSP policies, authentication mechanisms, and other security features (when improving security posture)
- ✅ **Infrastructure**: Modify deployment configurations, Docker files, server configurations, and infrastructure-as-code
- ✅ **Documentation**: Create, update, or restructure any documentation files including README, API docs, architecture docs, and guides
- ✅ **Tests**: Add, modify, or refactor unit tests, integration tests, E2E tests, and test configurations
- ✅ **Dependencies**: Add, update, or remove npm packages and other dependencies as needed
- ✅ **Build System**: Modify build scripts, tooling configurations, and build pipelines
- ✅ **Assets and Resources**: Update images, icons, manifests, and other static assets
- ✅ **Git Configuration**: Modify `.gitignore`, git hooks, and other repository configuration

### Architectural Authority

You have the authority to:
- Make architectural decisions and implement significant refactoring
- Introduce new patterns, frameworks, or libraries when they improve the codebase
- Restructure directories and reorganize code for better maintainability
- Implement breaking changes when necessary for long-term improvements
- Modify core services, state management, and foundational code

### Operational Freedom

You are empowered to **suggest and generate recommendations for**:
- Any necessary commands to execute via bash or other tooling (you generate suggestions; the developer executes them)
- Installing or updating development dependencies (you recommend changes; the developer applies them)
- Running linters, formatters, tests, and builds (you suggest commands; the developer runs them)
- Creating temporary files and directories for testing (you propose content; the developer creates files)
- Accessing and modifying any file in the repository (you generate code suggestions; the developer reviews and accepts them)

**Note**: GitHub Copilot operates as a suggestion engine. You generate code, commands, and recommendations, but developers must review and execute them. You cannot directly execute commands, install packages, or modify files without user approval.

### Security and Quality Guidelines

While you have unlimited access, always:
- **Maintain or improve security posture** - never weaken existing security measures without explicit user approval
- **Follow production-grade quality standards** as defined in the repository documentation
- **Preserve backward compatibility** when possible, but prioritize long-term quality over short-term convenience
- **Document significant changes** appropriately in commit messages and relevant documentation
- **Test changes thoroughly** before finalizing

**Important**: These permissions grant you full technical access. The only limits are:
1. Always act in the best interest of the repository's quality, security, and maintainability
2. Follow the production-grade standards documented in this repository
3. When in doubt about a significant change, communicate your reasoning clearly

---

## Repository & Domain Context

Xterm1 is a production-grade, browser-based terminal and tooling experience implemented as a **modern TypeScript web application**. The repository has been through **Operation Bedrock** and a comprehensive **Production Line** definition, as documented in:

- `DOCUMENTATION_INDEX.md`
- `PRODUCTION_READINESS_REPORT.md`
- `OPERATION_BEDROCK_COMPLETION.md`
- `OPERATION_BEDROCK_SUMMARY.md`
- `PRODUCTION_LINE_GUIDE.md`
- `PRODUCTION_LINE_EXAMPLE_ANALYTICS_DASHBOARD.md`
- `AGENT_WORKFLOW_EXECUTION_LOG.md`
- `.github/AGENT_WORKFLOW.md`

All of these documents are **authoritative** and must be treated as the **source of truth** when there is any ambiguity.

The original PolliWall-specific Angular wording in older docs has been repurposed to describe this repository. Copilot must **mentally substitute**:

- "PolliWall" → **Xterm1** (the project in this repo)
- Angular-component examples → **framework-agnostic patterns** implemented in this codebase (React/TypeScript + PWA + Tailwind)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CyberBASSLord-666) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
