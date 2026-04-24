---
trigger: always_on
description: @./skills/godmode/SKILL.md
---

# Godmode for Gemini CLI

@./skills/godmode/SKILL.md
@./skills/principles/SKILL.md

## Verify Installation

After running the install script, confirm everything is set up correctly:

```bash
bash adapters/gemini/verify.sh
```

This checks that Gemini CLI can find the Godmode configuration, skill files, and agent definitions. If any check fails, the script reports what is missing and how to fix it.

## Tool Mapping

Gemini CLI tools map to Godmode skill references as follows:

| Skill Reference | Gemini Tool |
|---|---|
| Read | `read_file` |
| Write | `write_file` |
| Edit | `replace` |
| Bash | `run_shell_command` |
| Grep | `grep_search` |
| Glob | `glob` |
| TodoWrite | `write_todos` |
| Skill | `activate_skill` |

When a skill says "use Read to examine the file," use `read_file`. When it says "use Bash to run tests," use `run_shell_command`. Apply this mapping throughout.

## How to Use Skills

Godmode has **126 skills** and **7 subagents**. The orchestrator (`/godmode`) auto-detects which to invoke. Users can also invoke directly: `/godmode:skillname`.

**When a user invokes a skill** (e.g., `/godmode:secure`), read the full skill file before executing:
```
read_file("./skills/secure/SKILL.md")
```
Then follow that skill's workflow exactly.

**General pattern:**
```
read_file("./skills/<skill-name>/SKILL.md")
```

## Subagents (7 Built-in)

Godmode ships with 7 specialized agents. For complex tasks, use the planner to decompose, then dispatch builders in parallel.

| Agent | Role | Read agents/*.md for full instructions |
|---|---|---|
| **planner** | Decomposes goals → parallel tasks mapped to skills | `read_file("./agents/planner.md")` |
| **builder** | Implements a task following a skill workflow | `read_file("./agents/builder.md")` |
| **reviewer** | Reviews code for correctness + security | `read_file("./agents/reviewer.md")` |
| **optimizer** | Autonomous measure → modify → verify loop | `read_file("./agents/optimizer.md")` |
| **explorer** | Read-only codebase recon | `read_file("./agents/explorer.md")` |
| **security** | STRIDE + OWASP audit | `read_file("./agents/security.md")` |
| **tester** | TDD test generation | `read_file("./agents/tester.md")` |

Note: Gemini CLI does not support parallel subagent dispatch. Execute agent roles sequentially: plan → explore → build → review → optimize.

## Skill Catalog (126 skills)

| Skill | Description |
|---|---|
| a11y | Accessibility testing and auditing |
| agent | AI agent development |
| analytics | Analytics implementation |
| angular | Angular architecture |
| api | API design and specification |
| apidocs | OpenAPI/Swagger documentation generation |
| architect | Software architecture |
| auth | Authentication and authorization |
| automate | Task automation |
| backup | Backup and disaster recovery |
| build | Build and execution |
| cache | Caching strategy |
| changelog | Changelog and release notes management |
| chaos | Chaos engineering |
| chart | Data visualization |
| cicd | CI/CD pipeline design |
| cli | CLI tool development |
| comply | Compliance and governance |
| concurrent | Concurrency and parallelism |
| config | Environment and configuration management |
| cost | Cloud cost optimization |
| cron | Scheduled tasks and job queue management |
| crypto | Cryptography implementation |
| ddd | Domain-Driven Design |
| debug | Scientific debugging |
| deploy | Advanced deployment strategies |
| designsystem | Design system architecture |
| devsecops | DevSecOps pipeline |
| distributed | Distributed systems design |
| django | Django and FastAPI development |
| docker | Docker mastery |
| docs | Documentation generation and maintenance |
| e2e | End-to-end testing |
| edge | Edge computing and serverless |
| email | Email and notification systems |
| eval | AI/LLM evaluation |
| event | Event-driven architecture |
| experiment | A/B testing and statistical analysis |
| fastapi | FastAPI mastery |
| feature | Feature flags and gradual rollouts |
| finish | Branch finalization |
| fix | Autonomous error fixing |
| forms | Form architecture |
| ghactions | GitHub Actions workflow design and optimization |
| git | Advanced Git workflows |
| godmode | Orchestrator (auto-detect phase and route) |
| graphql | GraphQL API development |
| grpc | gRPC and Protocol Buffers |
| i18n | Internationalization and localization |
| incident | Incident response and post-mortem |
| infra | Infrastructure as Code |
| integration | Integration testing |
| k8s | Kubernetes and container orchestration |
| laravel | Laravel mastery |
| legacy | Legacy code modernization |
| lint | Linting and code standards |
| loadtest | Load testing and performance testing |
| logging | Logging and structured logging |
| micro | Microservices design and management |
| migrate | Database migration and schema management |
| migration | System migration |
| ml | ML development and experimentation |
| mlops | MLOps and model deployment |
| mobile | Mobile app development |
| monorepo | Monorepo management |
| network | Network and DNS |
| nextjs | Next.js mastery |
| node | Node.js backend development |
| nosql | NoSQL database design |
| notify | Push, SMS, and in-app notifications |
| npm | Package management |
| observe | Monitoring and observability |
| onboard | Codebase onboarding |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arbazkhan971/godmode](https://github.com/arbazkhan971/godmode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
