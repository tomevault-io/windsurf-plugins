---
trigger: always_on
description: Production-grade software project development skill for OpenClaw. Structured event auditing, pluggable skill system, security gate, WBS hash attestation, auto context injection. 生产级 AI 编程项目管理技能——事件流审计、可插拔技能系统、安全门禁、WBS 哈希认证、上下文自动注入。Use when starting new projects or executing multi-step development tasks.
---


# SPM — Super Project Manager

## Overview

SPM is a comprehensive skill for software project development in OpenClaw. It integrates:

- **Superpowers** (21 workflows): Design brainstorming, implementation planning, TDD, subagent-driven development, code review, systematic debugging, git worktrees, Ralph Loop auto-retry, hashline edit verification, comment checker, preemptive compaction, todo enforcement, deep context initialization, Prometheus interview mode, AST-Grep + LSP, and more
- **PM enhancements**: Soul-searching protocol, assumption documentation, safe sandbox (/freeze & /guard), three-tier quality gates, project scaffolding, deployment pipeline
- **WBS Executor**: Structured task ledger with exit criteria, evidence tracking, heartbeat logging, interruption recovery, delivery summary

**Core Philosophy:** SPM is an orchestrator, not a monolith. Each phase triggers the right workflow. The WBS task ledger is the single source of truth for tracking — now protected by hash attestation and auto-injected into context.

### 🆕 v3.0.0 New Features

| Feature | Description | Priority |
|---------|-------------|----------|
| **WBS Hash Attestation** | SHA-256 integrity protection; tampered ledgers auto-detected | 🔒 Security |
| **Hook Auto-Injection** | Active tasks auto-injected into context before every tool call | 🤖 Automation |
| **Session Recovery** | Auto-generated recovery reports from heartbeat logs | 🔄 Resilience |
| **Parallel Task Pointers** | `.active_ledger` symlink + `scripts/switch-ledger.sh` for multi-task isolation | 📂 Multi-task |
| **SPM Minimal Mode** | 5-rule lightweight mode for <10 task projects | 🏃 Quick Start |
| **Template/Script Separation** | Clear distinction: user-project files vs skill internals | 📋 UX |

## When to Use

- Starting a new software project from scratch
- Implementing complex multi-file features
- Any task that spans multiple steps or sessions
- Work that requires quality gates, code review, or TDD

**When NOT to use:**
- Single-line fixes or typo corrections
- Quick one-file changes with obvious scope
- Pure brainstorming without execution intent

## Architecture

```
┌──────────────────────────────────────────────────────────┐
│                      SPM ORCHESTRATOR                     │
│  SKILL.md — Detects task type → Routes to correct phase  │
└──────────────────────────────────────────────────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        ▼                     ▼                     ▼
┌───────────────┐   ┌───────────────┐   ┌───────────────┐   ┌──────────────────┐
│  PHASE 0      │   │  REQUIREMENT   │   │   PLANNING    │   │    EXECUTION     │
│ ───────────── │   │ ─────────────  │   │  ──────────   │   │   ────────────   │
│ • Deep Ctx    │   │ • Intent Class │   │ • Write Plan  │   │ • Git Worktree   │
│   Init        │   │ • Research 1st │   │ • WBS Ledger  │   │ • TDD Cycle      │
│ • context-map │   │ • Test Assess  │   │ • Review Plan │   │ • Subagent Dev   │
│   .md         │   │ • Scope Lock   │   │ • Dependencies│   │ • Parallel Tasks │
│               │   │ • Brainstorm   │   │               │   │ • Hashline Verify│
│               │   │ • Design Doc   │   │               │   │                  │
└───────┬───────┘   └───────┬───────┘   └───────┬───────┘   └────────┬─────────┘
        └───────────────────┼────────────────────┘
                            ▼
┌───────────────┐   ┌───────────────┐   ┌──────────────────┐
│    QUALITY     │   │   DELIVERY    │   │  TRACKING (ALL)  │
│ ─────────────  │   │  ──────────   │   │  ──────────────  │
│ • Verification │   │ • Finish Brch │   │ • WBS Ledger     │
│ • Code Review  │   │ • Deploy (opt)│   │ • Heartbeat Log  │
│ • 3-Tier Gates │   │ • Delivery    │   │ • State Tracking │
│ • Ralph Loop   │   │ • Cleanup     │   │ • Recovery       │
│ • Comment Chk  │   │               │   │ • Preemptive Cmp │
└───────┬───────┘   └───────┬───────┘   └──────────────────┘
        └───────────────────┼────────────────────┘
                            ▼
              ┌─────────────────────────┐
              │    SUBAGENT DISPATCH     │
              │  ─────────────────────  │
              │  Impl. Subagent         │
              │  Spec Reviewer          │
              │  Code Quality Reviewer  │
              │  Parallel Subagents     │
              │  Todo Enforcement Gate  │
              └─────────────────────────┘
```

## The Complete Lifecycle

```
┌────────────────────────────────────────────────────────────────────┐
│  PHASE 0: CONTEXT INIT [NEW]            PHASE 1: REQUIREMENT     │
│  ┌──────────────────────┐              ┌────────────────────┐    │
│  │ Deep Context Init    │──Auto──────▶ │ Intent Classify    │→  │
│  │ (context-map.md)     │              │ + Research 1st     │   │
│  └──────────────────────┘              │ + Test Assess      │   │
│                                         │ + Scope Lock       │   │
│                                         └─────────┬──────────┘   │

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zhbcher/openclaw-spm](https://github.com/zhbcher/openclaw-spm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
