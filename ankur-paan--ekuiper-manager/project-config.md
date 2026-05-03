---
trigger: always_on
description: 1. Identity & Communication
---

1. Identity & Communication

Tone: Maintain a technical, concise, and objective tone in all communications and outputs. Avoid casual language or personal opinions.

Efficiency: Omit unnecessary apologies, greetings, or meta-commentary. Focus responses on actionable content like code, execution logs, or direct answers to the query.

Documentation: Every exported function or module must include clear JSDoc/TSDoc comments. Comments should explain the why behind code decisions, not just the what. This aids future maintainers in understanding the purpose of code.

Jargon Clarity: Avoid using acronyms or project-specific jargon without definition on first use. Maintain a shared glossary for any specialized terms so that all team members (and the AI agent) have a common understanding.

2. Security & Boundaries

Scope Constraint: The AI agent is strictly forbidden from writing or modifying files outside the project’s workspace root (except for its own log files in ~/.gemini/antigravity/logs/). This prevents unintentional changes to the host system.

Credential Safety: Never hardcode API keys, passwords, or other secrets in code. If a secret is required, retrieve it securely (e.g., via environment variables or a secrets manager). If running interactively, prompt the user to provide the secret or reference a pre-defined placeholder (e.g., check for a .env.example file for guidance).

Execution Policy: Potentially dangerous commands (like those involving sudo, rm -rf /, or system-level configuration changes) require explicit user confirmation. Use an ASK_USER step before executing such commands. The agent should not run destructive or high-privilege operations autonomously.

Network Safety: If the agent needs to make network requests to domains that are not explicitly known or whitelisted, it must inform the user before execution. All external calls should be transparent to ensure security (e.g., no hidden HTTP requests to unknown servers).

Dependency Auditing: Before installing any npm or pip packages, check for known vulnerabilities in those packages. For Node.js, run an npm audit (or use npm audit --production for prod dependencies) and for Python use pip-audit. Address or flag any critical vulnerabilities. Additionally, pin package versions (at least pin the major version) in package.json or requirements files to prevent unexpected breaking changes.

Secrets Rotation: Implement automatic rotation for API keys, tokens, and other credentials with a maximum lifespan of 90 days. Document the rotation procedures (how to generate new keys, update them, etc.). This reduces the risk of leaked or old credentials being exploited.

Access Control: (Related to "only AI") Ensure that only the AI agent (or authorized automated processes) modifies code and configuration within the allowed scope. Human interventions should go through code reviews and version control. This guarantees that all changes are trackable and the AI’s activity remains within approved boundaries.

3. Coding Standards

Technology Stack: Adhere to the preferred stack. For front-end, use React (Next.js with the App Router) and TypeScript with strict settings. Styling should use Tailwind CSS for utility-first design. Back-end or scripting should use appropriate modern frameworks in the given language.

Animation Framework: Use Framer Motion for React UI animations and transitions. This ensures consistency and smooth user experience for interactive elements.

Programming Paradigm: Favor functional programming patterns over class-based designs, especially in React components. Use hooks and functional components rather than class components to align with modern React best practices.

Error Handling: Implement robust error handling. Use try/catch blocks where appropriate and provide meaningful error messages that aid debugging. Incorporate React error boundaries in the front-end to gracefully handle UI errors. Do not leave console.log statements in production code; instead, utilize a structured logging system (e.g., Winston or pino) to log important events or errors.

Testing: Every module or function must have a corresponding unit test. When you implement new code, create tests that cover expected behavior and edge cases. Aim for high test coverage to catch regressions early.

API Design (Contract-First): Before writing implementation code for any API (internal or external), define the API contract clearly. Document endpoints, request/response schemas, status codes, and required headers using an OpenAPI/Swagger specification. Have the team (or relevant reviewers) review and sign off on this API contract before proceeding to implementation. This ensures everyone agrees on the data structures and reduces rework.

Logging: Use structured logging with context. Every log entry should have a consistent format (e.g., JSON) and include context like timestamps and request identifiers (trace IDs) for correlating events across services. Do not log sensitive information (PII, credentials, etc.). Logs should be actionable and monitored for anomalies.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ankur-paan/ekuiper-manager](https://github.com/ankur-paan/ekuiper-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
