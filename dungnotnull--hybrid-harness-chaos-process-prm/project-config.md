---
trigger: always_on
description: **Project**: hybrid-harness-chaos-process-prm
---

# CLAUDE.md — hybrid-harness-chaos-process-prm (Pro Max)

## Project Identity

**Project**: hybrid-harness-chaos-process-prm
**Version**: 0.5.0
**Domain**: Platform Engineering — Continuous Delivery + Resilience Engineering + Security + Compliance + Adversarial Critique
**Audience**: Fullstack developers, SREs, Platform Engineers, DevOps practitioners, Security engineers
**AI Compatibility**: Claude Code (with plugin manifest), Codex, Gemini, GPT-4, and any instruction-following LLM agent

---

## What This Project Does

This repository defines a **37-skill Agile workflow** covering the complete SDLC from ideation to production operations, including adversarial critique:

| Domain | Purpose |
|---|---|
| **Foundation** | Orchestration, BA analysis, user flows, taste memory, progress tracking |
| **CI/CD Engineering** | Pipeline design, service onboarding, delegates, secrets, feature flags, templates, GitOps |
| **Security** | SAST, DAST, container scanning, dependency scanning, SBOM, supply chain security |
| **Testing** | CloakBrowser E2E, performance/load testing, visual regression |
| **Chaos Engineering** | Hypothesis-driven fault injection, blast radius control, steady state, game days |
| **Verification** | Continuous verification, observability, alerting, recommendations |
| **Governance** | OPA policies, cloud cost, release management, disaster recovery |
| **Learning** | Resilience scoring, postmortem RCA, compliance & audit |
| **Research** | Deep multi-source research, evidence synthesis, brainstorming debrief |
| **Optimization** | Latency, N+1, stress, atomicity, concurrency, security audit |
| **Documentation** | Technical specs, user flows, usage guides, README generation |
| **Adversarial Critique** | Devil's Advocate: stress-test decisions, detect fallacies, score arguments |

---

## Quick Reference: Skill Index (37 Skills)

| # | Skill | Phase | Triggers |
|---|---|---|---|
| 00 | Orchestrator | Foundation | workflow, start, orchestrate |
| 01 | BA Requirements | Foundation | analyze, PRD, requirements, spec |
| 01-1 | User Flow Writing | Foundation | user flow, journey map, edge case, sad path |
| 02 | Taste Memory | Foundation | preference, always, never, I prefer |
| 03 | Progress Tracker | Foundation | status, progress, where are we |
| 04 | Pipeline Design | CI/CD | pipeline, CI/CD, stages, deploy |
| 05 | Service Onboarding | CI/CD | onboard, new service, register |
| 06 | Delegate Management | CI/CD | delegate, agent, install |
| 07 | Secrets Management | CI/CD | secret, credential, vault |
| 08 | Feature Flags | CI/CD | feature flag, FF, rollout, toggle |
| 09 | Template Library | CI/CD | template, reusable, stage template |
| 10 | GitOps | CI/CD | GitOps, ArgoCD, sync, drift |
| 11 | Security Scanning | Security | SAST, vulnerability, CVE, SBOM, scan |
| 12 | CloakBrowser Testing | Testing | test, E2E, browser, a11y, visual |
| 13 | Performance Testing | Testing | load test, stress, benchmark, k6 |
| 14 | Experiment Design | Chaos | chaos experiment, fault, inject |
| 15 | Hypothesis Validation | Chaos | hypothesis, steady state, SLO |
| 16 | Blast Radius Control | Chaos | blast radius, scope, limit, abort |
| 17 | Steady State | Chaos | steady state, baseline, probe |
| 18 | Infrastructure Faults | Chaos | node drain, disk, CPU, EC2 stop |
| 19 | Application Faults | Chaos | pod delete, container kill, latency |
| 20 | Game Day Planning | Game Day | game day, chaos day, war game |
| 21 | CV Verification | Verify | continuous verification, canary |
| 22 | Observability | Verify | observability, dashboard, metrics |
| 23 | Alerting | Verify | alert, notify, recommend, remediate |
| 24 | Policy Governance | Govern | OPA, policy, governance, compliance |
| 25 | Cloud Cost | Govern | cost, budget, optimization, CCM |
| 26 | Resilience Scoring | Govern | resilience score, maturity, report |
| 27 | Postmortem | Learn | postmortem, RCA, learning, action |
| 28 | Release Management | Govern | release, deploy prod, change mgmt |
| 29 | Disaster Recovery | Govern | DR, failover, RTO, RPO, backup |
| 30 | Compliance & Audit | Learn | compliance, audit, SOC2, HIPAA, GDPR |
| 31 | Strategic Creator | Any | think bigger, brainstorm, propose, innovate, upgrade |
| 32 | Deep Research | Any | research this, find papers, literature review, evidence for |
| 33 | System Optimization | Any | optimize, latency, N+1, stress test, CCU, atomicity, race condition, security audit |
| 34 | Documentation Writing | Any | docs, README, user guide, technical spec, userflow, usage instructions, how to use |
| 35 | Devil's Advocate | Any | challenge this, stress-test, devil's advocate, find flaws, counter-argument, red team, critique |

---

## Complete Agile Workflow Map

`
PHASE 0: FOUNDATION
  s00 -> s01 -> s02 + s03 (parallel)

PHASE 1: PLANNING & REQUIREMENTS
  s01 (BA deep analysis -> PRD + ADRs + backlog)
  s01-1 (User Flow Writing -> journeys, edge cases, sad paths)

-- s31 (STRATEGIC CREATOR — callable at ANY phase) --
     ^v can be invoked before, during, or after any phase

-- s32 (DEEP RESEARCH — callable at ANY phase) --
     ^v evidence-grounded research + brainstorming debrief

-- s33 (SYSTEM OPTIMIZATION — callable at ANY phase) --

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dungnotnull/hybrid-harness-chaos-process-prm](https://github.com/dungnotnull/hybrid-harness-chaos-process-prm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-08 -->
