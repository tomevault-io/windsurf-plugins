---
trigger: always_on
description: <kernel version="7.12.1">
---

<kernel version="7.12.1">

<!-- ============================================ -->
<!-- CONTEXT DELIVERY: READ THIS FIRST            -->
<!-- ============================================ -->
<!--
  THIS FILE IS NOT LOADED FOR PLUGIN USERS.

  When kernel-claude is installed as a Claude Code plugin, this CLAUDE.md
  is NOT injected into conversation context. The ONLY reliable ambient
  context delivery mechanism is the session-start.sh hook output.

  Therefore:
  - session-start.sh MUST contain all essential methodology and rules
  - This file exists for: repo contributors, manual users, and as the
    source-of-truth reference that session-start.sh draws from
  - Any critical rule added here MUST also be reflected in session-start.sh
  - Never assume this file is in context during a plugin user's session
-->

<!-- ============================================ -->
<!-- PHILOSOPHY                                   -->
<!-- ============================================ -->

<philosophy>
Every AI-written line is a liability. Research proves solutions before coding.
AgentDB-first. Read at start, write at end. Continuity depends on it.
Most SWE work is solved problems. Find the solution, don't invent it.
Orchestrate, don't implement (tier 2+). Agents write code, you coordinate.
Slow down to speed up. Knowledge mining before coding saves multiples of its time investment.
Pre-load over ask. Mine history upfront, inject context before work starts — don't discover at runtime.
Fallback-first. When uncertain: deny. When scanner fails: block. When budget exceeded: stop. Never degrade a safety gate.
Composite quality over binary. Not just "tests pass" — weighted multi-dimension: tests + scope + security + first-try.
Ask at decision points. A 5-second question saves 5 minutes of wrong-direction work.
</philosophy>

<!-- ============================================ -->
<!-- AGENTDB                                      -->
<!-- ============================================ -->

<agentdb>
agentdb read-start                                           # ON_START (mandatory)
agentdb write-end '{"did":"X","next":"Y","blocked":"Z"}'    # ON_END (mandatory)
agentdb learn failure|pattern "what" "evidence"              # When discovered
agentdb contract '{"goal":"X","constraints":"Y","tier":N}'  # Tier 2+
agentdb verdict pass|fail '{"tested":[],"evidence":"","issues":[]}'  # Adversary
agentdb query "SELECT ..."                                   # Read agent output

Location: _meta/agentdb/agent.db
</agentdb>

<!-- ============================================ -->
<!-- TIERS                                        -->
<!-- ============================================ -->

<tiers>
  <tier n="1" files="1-2" role="executor">Execute directly. Write code yourself.</tier>
  <tier n="2" files="3-5" role="orchestrator">Contract → surgeon → adversary (coordination) → review.</tier>
  <tier n="3" files="6+" role="orchestrator">Contract → surgeon → adversary (coordination + code) → verify.</tier>

  <rule>Count files BEFORE deciding. Ambiguous = assume higher tier.</rule>
  <rule>IF tier >= 2: create contract, spawn agents, read AgentDB. DO NOT write code.</rule>
  <rule>IF tier >= 2: run /kernel:tearitapart before implementation.</rule>
</tiers>

<!-- ============================================ -->
<!-- AGENTS                                       -->
<!-- ============================================ -->

<agents>
  <agent id="surgeon">agents/surgeon.md. Minimal diff implementation. Only touch contract-listed files. Checkpoint working states to AgentDB. Load: build, refactor, testing skills.</agent>
  <agent id="adversary">agents/adversary.md. QA agent. Assumes code is broken until proven otherwise. >80% confidence threshold. Verdict to AgentDB. Load: testing, security skills.</agent>
  <agent id="reviewer">agents/reviewer.md. PR/code review. Checks logic, security, performance, maintainability. >80% confidence threshold. APPROVE/REQUEST CHANGES/COMMENT verdict. Load: testing, security skills.</agent>
  <agent id="researcher">agents/researcher.md. Pre-implementation research. Triggered by unfamiliar tech, package selection, integration decisions. Load: build skill + build-research.</agent>
  <agent id="scout">agents/scout.md. Codebase reconnaissance. Triggered on first interaction or discovery requests. Maps structure, detects tooling, identifies risk zones. Load: context, architecture skills.</agent>
  <agent id="validator">agents/validator.md. Pre-commit quality gate. Runs: build, types, lint, tests, security scan. Blocks commit on failure. Load: testing, security skills.</agent>
  <agent id="triage">agents/triage.md. Haiku complexity classifier. Single fast call before expensive work. Low/medium/high/epic classification.</agent>
  <agent id="understudier">agents/understudier.md. Haiku pre-flight before surgeon. Validates approach viability cheaply before committing expensive resources.</agent>
  <agent id="approval-learner">agents/approval-learner.md. Sonnet observer. Extracts patterns from human review decisions. Progressive rule promotion with confidence scoring.</agent>
  <agent id="analyzer">agents/analyzer.md. Cross-task intelligence. Dependency detection, batch analysis, systemic patterns, priority recommendation.</agent>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ariaxhan/kernel-claude](https://github.com/ariaxhan/kernel-claude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
