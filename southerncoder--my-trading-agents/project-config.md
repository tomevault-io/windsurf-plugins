---
trigger: always_on
description: **MANDATORY**: Do not create summary documents, status reports, or documentation files unless explicitly requested by the user.
---

# Copilot Instructions for TradingAgents

## Critical Rule: No Unsolicited Documentation Files

**MANDATORY**: Do not create summary documents, status reports, or documentation files unless explicitly requested by the user.

### Why This Matters

1. **Token Waste**: Creating unnecessary documentation consumes tokens without adding value
2. **Repository Clutter**: Unsolicited documents add noise to the codebase
3. **Redundant Information**: Most information belongs in code comments or commit messages
4. **User Autonomy**: Let the user decide when documentation is needed

### When to Create Documentation

- ✅ **User explicitly requests documentation** ("create a README", "document this feature")
- ✅ **User asks for explanation** that would benefit from a file (ask first: "Would you like me to create a documentation file for this?")
- ❌ **Never create summary documents** of completed work without being asked
- ❌ **Never create status reports** unless specifically requested
- ❌ **Never create "what we achieved today" documents** - use commit messages instead

### Correct Approach

```
User: "We just completed the market analyst integration"
Assistant: ✅ Acknowledges completion, offers to help with next steps
          ❌ Does NOT create "MARKET-ANALYST-STATUS.md" unprompted
```

**When in doubt, ask**: "Would you like me to create a documentation file for [topic]?"

## Critical Security Rule: Never Include Real Network Information

**MANDATORY**: Never include real IP addresses, usernames, passwords, client IDs, or other sensitive network information in source code, test files, documentation, or any repository files.

### Why This Matters

1. **Security Policy Violation**: Exposing internal network details creates security vulnerabilities
2. **Network Topology Exposure**: Real IPs reveal internal network structure
3. **Attack Surface**: Hardcoded credentials provide attack vectors for malicious actors
4. **Privacy Concerns**: Personal/internal information should never be public
5. **Professional Standards**: Public repositories must maintain security hygiene

### Correct Approach: Docker Secrets and Environment Variables

```python
# ✅ CORRECT - Use environment variables
REMOTE_LM_STUDIO_BASE_URL = os.getenv('REMOTE_LM_STUDIO_BASE_URL', 'http://localhost:1234')
NEO4J_URI = os.getenv('NEO4J_URI', 'bolt://localhost:7687')
REDDIT_CLIENT_ID = os.getenv('REDDIT_CLIENT_ID', 'your_client_id_here')

# ✅ CORRECT - Use localhost in examples
base_url = 'http://localhost:1234/v1'
neo4j_uri = 'bolt://localhost:7687'

# ✅ CORRECT - Use generic examples in documentation
# Example: Connect to your LM Studio server at http://your-server:1234
# Set REDDIT_CLIENT_ID=your_actual_client_id in .env.local
```

### INCORRECT Approach: Hardcoded Real Information

```python
# ❌ NEVER DO THIS - exposes real sensitive information
base_url = 'http://127.0.0.1:9876/v1'  # Real IP - SECURITY VIOLATION
reddit_client_id = 'abc123realid'      # Real client ID - SECURITY VIOLATION
username = 'real_username'              # Real username - SECURITY VIOLATION
```

### Security Best Practices

1. **Docker Secrets (Production)**: Use Docker secrets for containerized deployments
2. **Environment Variables (Development)**: Use .env.local for local testing (gitignored)
3. **Localhost Examples**: Use localhost or 127.0.0.1 in code examples
4. **Generic Documentation**: Use placeholders like "your-server", "your_client_id" in docs
5. **Secrets Migration**: Use `migrate-secrets.ps1` to sync root .env.local to Docker secrets
6. **Documentation Sanitization**: Replace real values with placeholders in all docs

### Dual Environment Architecture

**Docker Containers** (Production/Staging):
```
Root .env.local → migrate-secrets.ps1 → docker/secrets/*.txt → Docker Secrets (/run/secrets/) → init.js → Application
```

**Local Development** (Testing):
```
Root .env.local → migrate-secrets.ps1 → services/trading-agents/.env.local → dotenv → Application
```

**This rule applies to ALL files in the repository including source code, tests, documentation, and configuration examples.**

## File Organization Best Practices

**MANDATORY**: Follow established file organization patterns for maintainability and clarity.

### Component-Based Organization

When working with related functionality, organize files into component-specific folders:

```
docs/
├── reddit/           # Reddit integration documentation
├── zep-graphiti/     # Zep-Graphiti memory system docs
└── market-data/      # Market data provider documentation

services/trading-agents/src/
├── providers/
│   ├── reddit/           # Reddit-specific providers
│   ├── zep-graphiti/     # Zep-Graphiti memory providers
│   └── market-data/      # Market data providers
├── memory/
│   ├── advanced/         # Advanced memory implementations
│   └── providers/        # Memory provider interfaces
├── graph/                # LangGraph workflow implementations
├── cli/                  # CLI interface components
├── utils/                # Utility functions and helpers
└── tests/                # Test files

js/
├── src/
│   ├── providers/        # Legacy provider implementations
│   ├── graph/           # Graph workflow implementations

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [southerncoder/my-Trading-Agents](https://github.com/southerncoder/my-Trading-Agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
