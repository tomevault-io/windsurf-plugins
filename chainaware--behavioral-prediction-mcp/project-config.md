---
trigger: always_on
description: Machine-readable index of all 32 Claude Code subagents in `.claude/agents/`.
---

# ChainAware Subagent Index

Machine-readable index of all 32 Claude Code subagents in `.claude/agents/`.
Each agent is a specialist that handles a specific Web3 intelligence task using the
ChainAware Behavioral Prediction MCP (`https://prediction.mcp.chainaware.ai/sse`).

> 🏆 Featured in the [CB Insights Fraud Prevention Market Map for the AI Era](https://www.cbinsights.com/research/report/the-fraud-prevention-market-map-for-the-ai-era/) (2026)
> 🌐 Listed on the [BNB Chain AI Landscape](https://x.com/BNBCHAIN/status/1947597551139500419) (2025)
> 🚀 Listed on [BNB Chain Kickstart — Marketing Tools](https://www.bnbchain.org/en/programs/kickstart#services) (2025)
> 💰 Awarded a [$250k Google Cloud Grant](https://chainaware.ai/blog/google-cloud-grant/) (2025)
> ☁️ Accepted into the [AWS Fintech Accelerator](https://chainaware.ai/blog/aws-grant/) (2024)
> 🌱 Featured in the [Safary Club Web3 Growth Landscape — Growth Tools](https://x.com/Safaryclub/status/1822983239734329613) (2024)

**Links:** [Website](https://chainaware.ai) · [Twitter](https://x.com/ChainAware/) · [LinkedIn](https://www.linkedin.com/company/chainaware) · [Blog](https://chainaware.ai/blog) · [Learn](https://chainaware.ai/learn) · [Examples](https://github.com/ChainAware/examples)

**Accuracy:** [98% Fraud Detection](https://chainaware.ai/scam-db) · [90.1% Rug Pull Detection](https://chainaware.ai/resources/rugpull-verification) — backtesting verified

**Quick setup:**
```bash
claude mcp add --transport sse chainaware-behavioral-prediction \
  https://prediction.mcp.chainaware.ai/sse --header "X-API-Key: YOUR_KEY"
export CHAINAWARE_API_KEY="your-key-here"
```

---

## Agent Directory

### chainaware-wallet-auditor
**File:** `.claude/agents/chainaware-wallet-auditor.md`
**Model:** claude-sonnet-4-6
**Tools:** `predictive_behaviour`
**Purpose:** Full due diligence. Deep behavioural analysis of a wallet including fraud signals, intent, experience, and recommendations.
**Triggers:** "full audit of 0x...", "due diligence on this wallet", "complete analysis", "deep dive on this address"
**Input:** wallet address + network
**Output:** Behaviour profile, fraud signals, experience score, intent, recommendations

---

### chainaware-fraud-detector
**File:** `.claude/agents/chainaware-fraud-detector.md`
**Model:** claude-haiku-4-5-20251001
**Tools:** `predictive_fraud`
**Purpose:** Fast wallet fraud screening. Returns fraud status, probability, and forensic AML flags.
**Triggers:** "is this wallet safe?", "fraud check on 0x...", "AML screen", "is this address suspicious?", "check before I transact"
**Input:** wallet address + network
**Output:** Status (Fraud / Not Fraud / New Address), probabilityFraud (0–1), forensic flags

---

### chainaware-rug-pull-detector
**File:** `.claude/agents/chainaware-rug-pull-detector.md`
**Model:** claude-haiku-4-5-20251001
**Tools:** `predictive_rug_pull`, `predictive_fraud`
**Purpose:** Smart contract and LP safety checks before depositing funds.
**Triggers:** "will this pool rug pull?", "is this contract safe?", "check this LP", "rug pull risk", "should I ape in?"
**Input:** smart contract or LP address + network (ETH, BNB, BASE, HAQQ)
**Output:** Rug pull probability, risk tier, deployer risk, forensic breakdown, recommendation

---

### chainaware-wallet-marketer
**File:** `.claude/agents/chainaware-wallet-marketer.md`
**Model:** claude-sonnet-4-6
**Tools:** `predictive_behaviour`, `predictive_fraud`
**Purpose:** Generates a hyper-personalised marketing message (≤20 words) calibrated to the wallet's behaviour profile.
**Triggers:** "write a message for this wallet", "personalise outreach for 0x...", "convert this user", "marketing for this address"
**Input:** wallet address + network
**Output:** One personalised message (≤20 words) + rationale

---

### chainaware-reputation-scorer
**File:** `.claude/agents/chainaware-reputation-scorer.md`
**Model:** claude-haiku-4-5-20251001
**Tools:** `predictive_behaviour`
**Purpose:** Calculates a numeric reputation score using the ChainAware formula.
**Formula:** `(1000 / 110) × (experience + 1) × (riskCapability + 1) × (1 - probabilityFraud)` — max score = 1000; `experience` (0–10) and `riskCapability` (0–9) are direct fields from `predictive_behaviour`
**Triggers:** "reputation score for 0x...", "score this wallet", "rank these wallets", "which wallet is better?"
**Input:** wallet address + network
**Output:** Reputation score (0–1000), band label, component breakdown

---

### chainaware-aml-scorer
**File:** `.claude/agents/chainaware-aml-scorer.md`
**Model:** claude-haiku-4-5-20251001
**Tools:** `predictive_fraud`
**Purpose:** AML compliance scoring for KYC, regulatory, and exchange onboarding workflows.
**Formula:** `(1 - probabilityFraud) × 100` — returns 0 if any forensic flag is present
**Triggers:** "AML score for 0x...", "is this wallet AML compliant?", "KYC screening", "compliance check", "run AML check"
**Input:** wallet address + network
**Output:** AML score (0–100), pass/fail verdict, forensic flag breakdown

---

### chainaware-trust-scorer
**File:** `.claude/agents/chainaware-trust-scorer.md`
**Model:** claude-haiku-4-5-20251001
**Tools:** `predictive_fraud`
**Purpose:** Returns a simple trust score as `1 - fraud_probability`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ChainAware/behavioral-prediction-mcp](https://github.com/ChainAware/behavioral-prediction-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-03 -->
