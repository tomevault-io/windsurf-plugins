---
trigger: always_on
description: Agent Audit is a security scanner for AI agent code, MCP configurations, and DeFi contracts. It detects agent-specific vulnerabilities that traditional SAST tools miss, mapped to the OWASP Agentic Top 10 (2026) with 10/10 coverage.
---

# Agent Audit (Argus) — Project Context for Claude Code

## Overview

Agent Audit is a security scanner for AI agent code, MCP configurations, and DeFi contracts. It detects agent-specific vulnerabilities that traditional SAST tools miss, mapped to the OWASP Agentic Top 10 (2026) with 10/10 coverage.

- **Version**: 0.18.2
- **Python**: 3.9-3.12
- **License**: MIT
- **Entry point**: `agent-audit = "agent_audit.cli.main:cli"`
- **Metrics**: 94.6% recall, 87.5% precision, F1=0.91, 1239+ tests

## Repository Structure

```
agent-security-suite/
├── packages/audit/                    # Main Python package
│   ├── agent_audit/
│   │   ├── __init__.py
│   │   ├── version.py                # __version__ = "0.18.2"
│   │   ├── analysis/                 # 21 analyzer modules (confidence scoring, taint, context)
│   │   │   ├── semantic_analyzer.py  # 3-stage credential detection (regex → value → context)
│   │   │   ├── taint_tracker.py      # Data flow: source → sanitizer → sink
│   │   │   ├── tool_boundary_detector.py  # @tool entry point gate for AGENT-034
│   │   │   ├── ts_tool_boundary_detector.py
│   │   │   ├── confidence_matrix.py  # Tier computation and adjustment rules
│   │   │   ├── context_classifier.py # File type detection (test/example/infra)
│   │   │   ├── dangerous_operation_analyzer.py
│   │   │   ├── framework_detector.py # Pydantic/LangChain/CrewAI internal suppression
│   │   │   ├── placeholder_detector.py
│   │   │   ├── value_analyzer.py
│   │   │   ├── identifier_analyzer.py
│   │   │   ├── tool_description_analyzer.py
│   │   │   ├── memory_method_detector.py
│   │   │   ├── env_tracer.py
│   │   │   ├── entropy.py
│   │   │   └── rule_context_config.py
│   │   ├── analyzers/
│   │   │   └── memory_context.py
│   │   ├── scanners/                 # 14 scanner modules
│   │   │   ├── base.py              # BaseScanner ABC
│   │   │   ├── python_scanner.py    # 4017 LOC — AST-based Python analysis
│   │   │   ├── typescript_scanner.py # 952 LOC — TS/JS via tree-sitter
│   │   │   ├── go_scanner.py        # 296 LOC — Go patterns
│   │   │   ├── solidity_scanner.py  # 411 LOC — Solidity contracts
│   │   │   ├── secret_scanner.py    # 855 LOC — Regex + semantic secret detection
│   │   │   ├── config_scanner.py    # 390 LOC — YAML/JSON/ENV files
│   │   │   ├── mcp_config_scanner.py # 1369 LOC — MCP config auditing
│   │   │   ├── mcp_baseline.py      # 530 LOC — MCP baseline drift (rug pull)
│   │   │   ├── mcp_inspector.py     # 607 LOC — Live MCP server introspection
│   │   │   ├── privilege_scanner.py # 1138 LOC — OS privilege escalation
│   │   │   ├── skill_body_scanner.py # 402 LOC — OpenClaw skill instructions
│   │   │   ├── skill_meta_scanner.py # 352 LOC — OpenClaw skill metadata
│   │   │   └── __init__.py
│   │   ├── rules/
│   │   │   ├── engine.py            # RULE_CWE_MAPPING (109 rules), PATTERN_TYPE_TO_RULE_MAP (55+ patterns)
│   │   │   ├── loader.py            # YAML rule loader
│   │   │   └── builtin/             # YAML rule definitions (mirrored from monorepo)
│   │   │       ├── owasp_agentic_v2.yaml
│   │   │       ├── owasp_agentic.yaml
│   │   │       ├── asi_coverage_v030.yaml
│   │   │       ├── mcp_security_v030.yaml
│   │   │       └── langchain_security_v030.yaml
│   │   ├── models/
│   │   │   ├── finding.py           # Finding dataclass, confidence_to_tier(), TIER_THRESHOLDS
│   │   │   ├── risk.py              # Severity/Category enums, Location, RiskScore
│   │   │   ├── suppression.py       # Suppression/ignore model
│   │   │   └── tool.py              # ToolDefinition, PermissionType
│   │   ├── cli/
│   │   │   ├── main.py              # @click group
│   │   │   ├── commands/
│   │   │   │   ├── scan.py          # Main scan orchestration
│   │   │   │   ├── inspect.py       # Live MCP inspection
│   │   │   │   └── init.py          # Config init
│   │   │   └── formatters/
│   │   │       ├── terminal.py      # Rich terminal output + calculate_risk_score()
│   │   │       ├── json.py          # JSON output
│   │   │       └── sarif.py         # SARIF v2.1.0 output
│   │   ├── profiles/
│   │   │   └── defi/                # DeFi profile (AGENT-090 to AGENT-109)
│   │   │       ├── rules.py
│   │   │       ├── scanners/        # solidity, js_ts, defi_secret, agent_payment
│   │   │       ├── analysis/        # llm_analyzer, rpc_analyzer, defi_taint
│   │   │       └── constants/       # defi_protocols, web3_apis, rpc_endpoints
│   │   ├── config/
│   │   │   └── ignore.py            # .agent-audit.yaml loader
│   │   ├── utils/
│   │   │   ├── mcp_client.py
│   │   │   └── compat.py
│   │   └── parsers/
│   │       └── treesitter_parser.py
│   ├── tests/                        # 1239+ tests
│   │   ├── test_agent004_semantic.py
│   │   ├── test_expanded_rules.py
│   │   ├── test_privilege_rules.py
│   │   ├── test_defi_profile.py
│   │   ├── test_go_scanner.py
│   │   ├── test_analysis/            # 17 analyzer test modules
│   │   ├── test_cli/                 # 5 CLI test modules
│   │   ├── test_formatters/          # 4 formatter test modules

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HeadyZhang/agent-audit](https://github.com/HeadyZhang/agent-audit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
