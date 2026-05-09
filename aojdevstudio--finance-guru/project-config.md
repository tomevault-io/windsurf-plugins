---
trigger: always_on
description: [Skills Index]|root: ./.claude/skills|Mirrors: ./.agents/skills (symlinked per-skill), ./.pi/skills (top-level symlink → .agents/skills) — makes Finance Guru skills portable to pi-coding-agent and any Agent Skills-standard harness. See docs/reference/cross-harness-skills.md|IMPORTANT: Read full SKILL.md before using any skill. This index is for routing only.|backend-dev-guidelines:{name:backend-dev-guidelines,desc:Comprehensive backend development guide for Node.js/Express/TypeScript microservic
---



[Skills Index]|root: ./.claude/skills|Mirrors: ./.agents/skills (symlinked per-skill), ./.pi/skills (top-level symlink → .agents/skills) — makes Finance Guru skills portable to pi-coding-agent and any Agent Skills-standard harness. See docs/reference/cross-harness-skills.md|IMPORTANT: Read full SKILL.md before using any skill. This index is for routing only.|backend-dev-guidelines:{name:backend-dev-guidelines,desc:Comprehensive backend development guide for Node.js/Express/TypeScript microservices.,files:{resources:{architecture-overview.md,async-and-errors.md,complete-examples.md,configuration.md,database-patterns.md,middleware-guide.md,routing-and-controllers.md,sentry-and-monitoring.md,services-and-repositories.md,testing-guide.md,validation-patterns.md}}}|dividend-tracking:{name:dividend-tracking,desc:Sync dividend data from Fidelity CSV to Dividends sheet.,files:{}}|error-tracking:{name:error-tracking,desc:Add Sentry v8 error tracking and performance monitoring to your project services.,files:{}}|fin-core:{name:fin-core,desc:| Finance Guru™ Core Context Loader Auto-loads essential Finance Guru system configuration and user profile at session s,files:{README.md}}|FinanceReport:{name:FinanceReport,desc:Generate institutional-quality PDF analysis reports for stocks and ETFs.,files:{StyleGuide.md,VisGuide.md,tools:{ChartKit.help.md,ChartKit.py,ReportGenerator.help.md,ReportGenerator.py},workflows:{FullResearchWorkflow.md,GenerateSingleReport.md,RegenerateBatch.md}}}|formula-protection:{name:formula-protection,desc:Prevent accidental modification of sacred spreadsheet formulas in Google Sheets Portfolio Tracker.,files:{}}|margin-management:{name:margin-management,desc:Update Margin Dashboard with Fidelity balance data and calculate margin-living strategy metrics.,files:{}}|MonteCarlo:{name:MonteCarlo,desc:Run Monte Carlo simulations for Finance Guru portfolio strategy.,files:{PortfolioParser.md,tools:{.gitkeep},workflows:{IncorporateBuyTicket.md,RunSimulation.md}}}|PortfolioSyncing:{name:PortfolioSyncing,desc:Import and sync broker CSV portfolio data to Google Sheets DataHub.,files:{workflows:{SyncPortfolio.md}}}|python-performance-optimization:{name:python-performance-optimization,desc:Profile and optimize Python code using cProfile, memory profilers, and performance best practices.,files:{}}|readiness-report:{name:readiness-report,desc:Evaluate how well a codebase supports autonomous AI development.,files:{references:{criteria.md,maturity-levels.md},scripts:{analyze_repo.py,generate_report.py}}}|retirement-syncing:{name:retirement-syncing,desc:Sync retirement account data from Vanguard and Fidelity CSV exports to Google Sheets DataHub.,files:{}}|route-tester:{name:route-tester,desc:Test authenticated routes in the your project using cookie-based authentication.,files:{}}|TransactionSyncing:{name:TransactionSyncing,desc:Import Fidelity transaction history CSV into Google Sheets with smart categorization.,files:{CategoryRules.md,workflows:{SyncTransactions.md}}}|[14 skills, 32 files]



## Project Overview

This is **Finance Guru™** - a private AI-powered family office system built on BMAD-CORE™ v6 architecture. This repository serves as the operational center for a multi-agent financial intelligence system that provides research, quantitative analysis, strategic planning, and compliance oversight.

**Key Principle**: This is NOT a software product or app - this IS Finance Guru, a personal financial command center working exclusively for the user. All references should use "your" when discussing assets, strategies, and portfolios.

# Architecture

### Multi-Agent System

Finance Guru™ uses a **specialized agent architecture** where Claude transforms into different financial specialists:

**Primary Entry Point**:

- **Finance Orchestrator** (Cassandra Holt) - Master coordinator located at `.claude/commands/fin-guru/agents/finance-orchestrator.md`

## Path Variable System

The codebase uses a variable substitution system:

- `{project-root}` - Root of the repository
- `{module-path}` - Path to fin-guru module
- `{current_datetime}` - Current date and time
- `{current_date}` - Current date (YYYY-MM-DD)
- `{user_name}` - User's name from config

When referencing files in agent configurations, these variables should be resolved to actual paths.

## External Tool Requirements

Finance Guru requires these MCP servers:

- **exa** - Deep research and market intelligence
- **bright-data** - Web scraping (search engines, markdown extraction)
- **sequential-thinking** - Complex multi-step reasoning
- **financial-datasets** - SEC filings, financial statements
- **gdrive** - Google Drive integration (sheets, docs)
- **web-search** - Real-time market information

## Temporal Awareness


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AojdevStudio/Finance-Guru](https://github.com/AojdevStudio/Finance-Guru) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
