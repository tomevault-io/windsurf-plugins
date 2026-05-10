---
trigger: always_on
description: Agent swarm system for analyzing call transcripts, CRM data, and any structured dataset using parallel Claude agents. Built on Blueprint GTM methodology by Jordan Crawford (blueprintgtm.com).
---

# Blueprint Swarm — System Instructions

## What This Is

Agent swarm system for analyzing call transcripts, CRM data, and any structured dataset using parallel Claude agents. Built on Blueprint GTM methodology by Jordan Crawford (blueprintgtm.com).

No API keys required. Pure LLM analysis on local files. Sonnet agents for extraction, Opus for quality audits.

## Blueprint GTM Methodology

This tool implements the Blueprint GTM framework for customer intelligence. Key principles that agents MUST follow:

- **Pain-qualified segmentation**: Focus on specific pains that predict buying behavior, not demographic proxies. "What changed in the customer's situation that means they need us now?"
- **Specificity breeds trust**: Every finding must include verbatim quotes traced to specific records. No paraphrasing, no editorializing.
- **Data over claims**: Show the evidence. If a pattern appeared in 47 out of 89 accounts, say "47/89 (53%)" — not "most" or "many."
- **Source-tagged everything**: Every insight has a provenance trail. Account name, call date, speaker role.

These are not decorations — they are the analytical framework. Agents that don't follow them produce lower-quality output.

## Critical Rules

### Data Safety
- NEVER modify the user's source data. Read only.
- All outputs go to `Blueprint-Swarm/data/{run-id}/`
- Each run gets a unique run-id (ISO timestamp: `2026-03-27T15-33-00`)
- Normalized copies of source data go to `data/{run-id}/normalized/`

### Agent Rules
- Sub-agents NEVER generate IDs. Key by filename or natural key.
- Sub-agents NEVER modify source files. Write only to their output path.
- Opus audit is MANDATORY after 5+ parallel agents.
- Reasoning vs tools: classification = reasoning only. No tools beyond file I/O.
- Default model: **Sonnet** for all analysis agents. **Opus** for auditor only.

### API Prohibition (CRITICAL)

Blueprint Swarm uses ONLY Claude Code subagents (the Agent tool). This is non-negotiable.

**PROHIBITED — any of the following is a critical violation:**
- `import anthropic` or `from anthropic import ...` — NEVER use the Anthropic SDK
- `anthropic.Anthropic(api_key=...)` — NEVER instantiate an API client
- `claude --print` or `claude -p` — uses API tokens, NOT Claude Code tokens
- `subprocess.run(["claude", ...])` — spawning Claude via CLI subprocess
- Any HTTP request to `api.anthropic.com` — direct API calls
- Any reference to `ANTHROPIC_API_KEY` environment variable for LLM inference

**REQUIRED — the ONLY way to launch analysis agents:**
- The `Agent` tool within Claude Code (subagent_type: "general-purpose")
- Agent Teams if `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` is enabled

If you encounter a Python script (e.g., `13_run_agent_swarm.py`, `23_run_swarm_v2.py`) that uses the Anthropic SDK or `claude --print`, DO NOT EXECUTE IT. Instead, inform the user that the script uses API tokens and offer to run the analysis through the Claude Code Agent tool instead.

**Rationale**: The user pays for Claude Code tokens through their subscription. API calls consume separate credits. Blueprint Swarm must only use what the user is already paying for.

### Rate Limit Protocol

All subagents share the orchestrator's Claude Code rate limit pool. See `references/rate-limit-protocol.md` for the full protocol.

**Key rules:**
- **Safe concurrency: 6-10 agents** — tested safe for sustained batch processing
- **Danger zone: 15+ agents** — triggers cascading 429s and usage cap hits
- **Never exceed 12 concurrent** — even briefly, the rate limit pool is shared across all agents
- Run continuously — no artificial session budget cap
- Launch new waves of 3 as previous agents complete (don't wait for all to finish)
- If rate-limited: state is auto-saved via StopFailure hook (`hooks/swarm-stop-failure.js`)
- Watchdog script (`scripts/swarm-watchdog.sh`) auto-resumes after the rate limit window resets
- On resume: read `data/{run-id}/state.json`, continue from next pending wave
- State is updated after every wave — nothing is lost on interruption

**CRITICAL — Overnight / Unattended Runs:**
The watchdog is NOT optional for overnight runs. Without it, the swarm dies at the first usage cap and sits dead until the user manually resumes — potentially wasting 12+ hours. Before ANY unattended run:

1. Start Claude Code inside tmux: `tmux new-session -s swarm`
2. Split the pane: `Ctrl-B %`
3. Start the watchdog in the right pane: `bash Blueprint-Swarm/scripts/swarm-watchdog.sh`
4. Switch back to the left pane to run the swarm

The watchdog detects "out of extra usage" / "rate limit" messages, sleeps until the reset time, then sends a resume prompt automatically. This is the ONLY mechanism for unattended overnight recovery.

**Usage Cap vs Rate Limit — Two Different Failure Modes:**
- **Rate limit (429)**: Temporary, resets in minutes. Agents fail fast, retry in next wave.
- **Usage cap ("out of extra usage")**: Daily/plan limit. Resets at midnight or plan boundary. ALL agents fail. Requires waiting hours. The watchdog handles both.

### Quote Integrity
- "Verbatim quotes" means EXACT text from source records. Never paraphrase.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SantaJordan/blueprint-swarm](https://github.com/SantaJordan/blueprint-swarm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-28 -->
