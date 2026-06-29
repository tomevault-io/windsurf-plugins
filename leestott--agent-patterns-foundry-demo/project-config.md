---
trigger: always_on
description: This document describes every agent across all seven demos, the orchestration pattern each demo uses, and how to configure the model they run on.
---

# Agent Patterns Demo Pack — Agent Reference

This document describes every agent across all seven demos, the orchestration pattern each demo uses, and how to configure the model they run on.

---

## Model Configuration

All agents are powered by a single, runtime-switchable model provider. Choose between **Foundry Local** (on-device, no API key) or **Microsoft Foundry** (cloud). Switch providers and models live from the **⚙ Model Settings** panel in the launcher UI without restarting the app.

### Foundry Local (on-device)

Models run entirely on your device. The UI model picker shows every model in three states:

| Status | Meaning |
|--------|---------|
| **Loaded** | Model is currently in memory — fastest start |
| **Cached** | Downloaded to disk, will load in seconds |
| **Available** | In the catalog, will download on first use |

Cards show device type (GPU/CPU), size in MB, tool-calling support, publisher, and task type.

### Microsoft Foundry (cloud)

Enter your Foundry endpoint URL and API key. Click **List Models** to browse your deployed models and click any entry to select it. The **Deployment Name** field is optional — leave blank to use the model name as the deployment identifier.

### API Endpoints

| Endpoint | Description |
|----------|-------------|
| `GET /api/models/local` | Returns catalog models with live `loaded`/`cached`/`catalog` status |
| `GET /api/models/azure` | Lists deployed models from the configured Foundry endpoint |
| `GET /api/model-config` | Current provider, model, and endpoint settings |
| `POST /api/model-config` | Update provider/model settings (persists to `.env`) |

---

## Demo 1 — Maker-Checker PR Review

**Pattern:** Sequential — iterative review loop  
**Builder:** `SequentialBuilder`  
**File:** [demos/maker_checker/agents.py](demos/maker_checker/agents.py)

### Agents

| Agent | Role | Behaviour |
|-------|------|-----------|
| **Worker** | Drafter | "You are a senior developer. Draft a concise PR review addressing code quality, bugs, and suggestions." Produces the initial review. |
| **Reviewer** | Quality gate | "You are a code review lead. Critique the draft against: correctness, clarity, actionability. Score 1-5." If score < 4, Worker revises. |

### Topology

```
Worker ──draft──► Reviewer ──score < 4?──► Worker (revision)
                     │
                  score ≥ 4
                     ▼
                    Done
```

### When to use this pattern

Quality gates, approval steps, iterative document drafting, compliance checking, code review automation. Any task where the first attempt rarely meets the bar and a structured feedback loop improves quality.

---

## Demo 2 — Hierarchical Research Brief

**Pattern:** Concurrent fan-out + Sequential synthesis  
**Builder:** `ConcurrentBuilder` (specialists) + `SequentialBuilder` (synthesis)  
**File:** [demos/hierarchical_research/agents.py](demos/hierarchical_research/agents.py)

### Agents

| Agent | Role | Behaviour |
|-------|------|-----------|
| **Manager** | Decomposer | "Decompose the research topic into 2 specific sub-questions." Produces the task list. |
| **Specialist_A** | Technical researcher | "Research the technical aspects of the given topic." Runs in parallel with Specialist_B. |
| **Specialist_B** | Market researcher | "Research the market/business aspects of the given topic." Runs in parallel with Specialist_A. |
| **Synthesizer** | Report writer | "Combine the specialist reports into a cohesive 200-word brief." Runs after both specialists complete. |

### Topology

```
Manager ──► Specialist_A ──┐
        └──► Specialist_B ──┴──► Synthesizer ──► Done
```

### When to use this pattern

Multi-source research, technical due diligence, market analysis, any task where independent sub-questions can be explored in parallel and merged into a single output.

---

## Demo 3 — Hand-off Customer Support

**Pattern:** Explicit agent-to-agent control transfer  
**Builder:** `HandoffBuilder`  
**File:** [demos/handoff_support/agents.py](demos/handoff_support/agents.py)

### Agents

| Agent | Role | Behaviour |
|-------|------|-----------|
| **Triage** | Classifier | "Classify customer issue as BILLING or TECH. Hand off to the right specialist." Emits a `handoff` event. |
| **Billing** | Billing specialist | "Handle billing inquiries: refunds, charges, payment methods." Receives control from Triage when issue is billing-related. |
| **TechSupport** | Tech specialist | "Handle technical issues: connectivity, errors, setup." Receives control from Triage when issue is technical. |

### Topology

```
Triage ──BILLING──► Billing ──► Done
       └──TECH────► TechSupport ──► Done
```

### When to use this pattern

Helpdesk automation, intent-based routing, HR query routing, escalation pipelines. Use when control must transfer cleanly to a specialist without a shared conversation buffer.

---

## Demo 4 — Network Brainstorm

**Pattern:** Peer group chat — all agents share one conversation thread  
**Builder:** `GroupChatBuilder`  
**File:** [demos/network_brainstorm/agents.py](demos/network_brainstorm/agents.py)

### Agents

| Agent | Role | Behaviour |
|-------|------|-----------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [leestott/agent_patterns_foundry_demo](https://github.com/leestott/agent_patterns_foundry_demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
