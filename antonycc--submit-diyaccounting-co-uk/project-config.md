---
trigger: always_on
description: handles execution and testing.
---

# DIY Accounting Submit - GitHub Copilot Code Review Instructions

**Last Updated:** 2026-01-05

## About This File

This file contains guidelines for **GitHub Copilot** code review agent. The repository also has guidelines for other AI coding assistants:
- `CLAUDE.md` + `.claude/rules/` - Guidelines for Claude Code (emphasis on autonomous task execution & implementation)
- `.junie/guidelines.md` - Guidelines for Junie (custom agent, emphasis on testing & iteration)

Each assistant has complementary strengths - GitHub Copilot is optimized for code review, analysis, and providing thoughtful feedback.

## Purpose

These instructions guide GitHub Copilot's code review agent to provide specialized, high-quality reviews for this repository. The focus is on **analysis and understanding** rather than test execution.

## Repository Documentation

**Primary Reference**: See [`./REPORT_REPOSITORY_CONTENTS.md`](../REPORT_REPOSITORY_CONTENTS.md) for comprehensive technical documentation including:

- Complete architecture overview (AWS serverless stack)
- All npm scripts in `package.json` with detailed descriptions
- Maven/CDK build process and stack organization
- Environment configuration (`.env.test`, `.env.ci`, `.env.proxy`, `.env.prod`)
- Testing strategy (unit, system, browser, behaviour)
- GitHub Actions CI/CD workflows
- Directory structure and file purposes
- AWS deployment architecture and security model

**When reviewing code**, reference REPORT_REPOSITORY_CONTENTS.md to understand context, verify script usage, and check consistency with documented patterns.

## Code Review Philosophy

### Favor Analysis Over Execution

As a code review agent, prioritize **static analysis and code comprehension** over running tests:

1. **Read and understand** code paths by tracing through files
2. **Mentally dry-run** logic to identify potential issues
3. **Validate consistency** with existing patterns and conventions
4. **Check references** against documented scripts and configuration
5. **Suggest tests** when appropriate, but let developers/CI run them

**Note**: The `.junie/guidelines.md` file describes behavior for the Junie custom agent, which emphasizes continuous
testing and iteration. As a code review agent, your role is complementary - you provide thoughtful analysis while Junie
handles execution and testing.

### Analysis Workflow

When reviewing code changes be as low friction as possible, maintaining current standards for the incoming code.

1. **Understand the context**
   - What problem is being solved?
   - Which components/files are affected?
   - What are the environmental implications?

2. **Trace code paths**
   - Follow execution flow through functions/modules
   - Identify dependencies and side effects
   - Check error handling and edge cases

3. **Validate against patterns**
   - Does it match existing code style?
   - Are naming conventions consistent?
   - Is it using documented npm scripts correctly?

4. **Consider cross-cutting concerns**
   - Security implications (secrets, IAM, input validation)
   - Performance and cost (Lambda execution, DynamoDB queries)
   - Testing coverage (are appropriate tests included?)
   - Environmental differences (test vs. CI vs. prod)

5. **Suggest minimal changes**
   - Focus on surgical, targeted improvements
   - Preserve working code unless fixing security issues
   - Match local style over enforcing global style rules
   - Do not raise issues where new possibly problematic code has been introduced but this pattern is already established
   - Do not raise issues about unused imports or variables or formatting
   - Do do raise concerns about performance where the suggested optimisation is unlikely to be a measurable signal above the noise.

## Repository Patterns and Conventions

### Code Style and Formatting

**JavaScript/TypeScript** (ES Modules):
- **Linter**: ESLint with flat config (`eslint.config.js`)
- **Formatter**: Prettier (`.prettierrc`)
- **Scripts**: Only run if specifically asked to fix formatting and linting errors: `npm run linting`, `npm run linting-fix`, `npm run formatting`, `npm run formatting-fix`
- **Convention**: Only run if specifically asked to fix formatting and linting errors: `npm run linting-fix && npm run formatting-fix`

**Java** (AWS CDK Infrastructure):
- **Formatter**: Spotless with Palantir Java Format (100-column width)
- **Scripts**: Only run if specifically asked to fix formatting and linting errors: `./mvnw spotless:check`, `./mvnw spotless:apply`
- **Convention**: Runs during Maven `install` phase, fails build if not formatted

**General Style Rule**: Match existing local style rather than forcing global rules when it would be disruptive. Only change style in code you're already modifying.

Avoid unnecessary formatting changes when editing code.
For the lines that you change, be compliant with the formatting rules.
Do not run formatting tools on the whole repository or whole files unless the whole file is new.

### Testing Strategy

**Test**: Run the following test commands in sequence to check that the code works:
```
npm test
./mvnw clean verify
npm run test:submitVatBehaviour-proxy
```
If you need to capture the output of a test do it like this:
```
npm test > target/test.txt 2>&1
./mvnw clean verify > target/mvnw.txt 2>&1

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/antonycc) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
