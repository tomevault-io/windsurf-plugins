---
trigger: always_on
description: Commercial real estate lease analysis toolkit: abstraction, financial analysis, and rental yield curves.
---

# CLAUDE.md

Commercial real estate lease analysis toolkit: abstraction, financial analysis, and rental yield curves.

## Structure

```
├── Shared_Utils/           # Shared financial utilities (NPV, IRR, PV, ratios, statistics) + schemas/
├── Eff_Rent_Calculator/    # Effective rent, NPV, breakeven analysis
├── Rental_Yield_Curve/     # Term structure pricing using implied termination options
├── Rental_Variance/        # Rental variance decomposition (rate, area, term)
├── IFRS16_Calculator/      # IFRS 16/ASC 842 lease accounting (liability, ROU asset, schedules)
├── Credit_Analysis/        # Tenant credit scoring and financial analysis
├── Comparable_Sales_Analysis/  # Traditional DCA comparable sales (dollar adjustments)
├── Renewal_Analysis/       # Renewal vs. relocation economic analysis
├── Option_Valuation/       # Real options valuation (Black-Scholes) for lease flexibility
├── Rollover_Analysis/      # Portfolio lease expiry and renewal prioritization
├── Default_Calculator/     # Tenant default damage quantification
├── Relative_Valuation/     # MCDA competitive positioning (25 variables) + statistical analysis
├── MCDA_Sales_Comparison/  # MCDA ordinal ranking for fee simple valuation (74 tests)
├── MLS_Extractor/          # MLS PDF to Excel extraction with subject highlighting
├── Planning/               # Source lease documents
├── Templates/              # Industrial/Office lease templates (24 sections, MD/JSON/Schema)
├── Reports/                # Generated abstracts & analysis (timestamp prefix required)
└── .claude/                # Claude Code configuration
    ├── commands/           # Slash commands (25 commands in 6 categories)
    │   ├── Abstraction/        # abstract-lease, critical-dates
    │   ├── Financial_Analysis/ # effective-rent, renewal-economics, tenant-credit, option-value, rental-variance, etc.
    │   ├── Accounting/         # ifrs16-calculation
    │   ├── Comparison/         # compare-amendment, compare-offers, compare-precedent, lease-vs-lease
    │   ├── Compliance/         # assignment-consent, default-analysis, estoppel-certificate, etc.
    │   └── Utilities/          # convert-to-pdf
    ├── skills/             # Expert skills (23 specialized skills - auto-invoked)
    │   ├── Core: commercial-lease-expert/
    │   ├── Financial Analysis: effective-rent-analyzer/, tenant-credit-analyst/,
    │   │                       lease-abstraction-specialist/
    │   ├── Compliance: lease-compliance-auditor/, default-and-remedies-advisor/,
    │   │               lease-comparison-expert/
    │   ├── Portfolio: portfolio-strategy-advisor/, real-options-valuation-expert/
    │   ├── Security: indemnity-expert/, non-disturbance-expert/
    │   ├── Transfers: consent-to-assignment-expert/, consent-to-sublease-expert/,
    │   │             share-transfer-consent-expert/, lease-surrender-expert/
    │   ├── Preliminary: offer-to-lease-expert/, waiver-agreement-expert/,
    │   │                temporary-license-expert/, storage-agreement-expert/
    │   ├── Specialized: telecom-licensing-expert/
    │   ├── Dispute: lease-arbitration-expert/
    │   └── Negotiation: negotiation-expert/, objection-handling-expert/
    ├── hooks/              # Intelligent skill activation (UserPromptSubmit + PreToolUse)
    │   ├── skill-activation-prompt.sh/ts     # Reactive: keyword-based skill suggestions
    │   ├── pre-tool-use-skill-loader.sh/ts   # Proactive: document type detection
    │   ├── generate-skill-rules.js           # Auto-generate activation rules from skills
    │   ├── lease-types-map.json              # Document type → skills mapping
    │   └── skill-rules.json                  # Auto-generated activation triggers (23 skills)
    └── agents/             # Sub-agents (The Triumvirate)
        ├── adam            # Adam - Senior Analyst (Haiku) - Fast execution for straightforward tasks
        ├── reggie-chan-vp  # Reggie Chan, CFA, FRICS - VP (Sonnet) - Complex problems & crisis management
        └── dennis          # Dennis - Strategic Advisor (Opus) - Strategic wisdom & reality checks
```

## Meet Your Team: The Triumvirate

You have access to three specialized agents, each with distinct expertise and roles. Together they form a complete professional support system.

### Adam - Your Everyday Analyst (Haiku Model)

**Senior Analyst** trained by Reggie Chan to handle straightforward tasks with institutional-grade rigor at exceptional speed.

**When to use Adam:**
- Standard lease evaluations (typical terms, normal tenants)
- Routine tenant credit checks (clear financials, no fraud concerns)
- Renewal offer assessments (clear market conditions)
- Simple deal comparisons (straightforward tradeoffs)
- Professional communication to stakeholders

**What Adam provides:**
- Fast execution (80/20 analysis)
- Reggie's analytical methods applied to day-to-day work
- Diplomatic delivery (politically aware communication)
- Quantified analysis without over-engineering

**Example interactions:**
```
"Adam, analyze this renewal offer at $25/sf with 3 months free rent"
"Adam, evaluate this tenant's credit - they're showing 1.3x DSCR"
"Adam, compare these two lease offers and recommend which is better"
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [reggiechan74/vp-real-estate](https://github.com/reggiechan74/vp-real-estate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
