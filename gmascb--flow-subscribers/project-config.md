---
trigger: always_on
description: This is a Ruby gem that implements a **Flow Pattern** for organizing backend code following **SOLID principles**.
---

# Flow Subscribers - Cursor Rules

## Overview

This is a Ruby gem that implements a **Flow Pattern** for organizing backend code following **SOLID principles**.

---

## Architecture

**FlowSubscribers** implements two flow patterns to organize business logic:

### Class Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              MODULE Flows                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────────────────┐    ┌─────────────────────────────────────┐ │
│  │      SIMPLE FLOW            │    │         COMPLETE FLOW               │ │
│  │                             │    │                                     │ │
│  │  ┌───────────────────────┐  │    │  ┌─────────────────────────────┐    │ │
│  │  │ SimpleFlowController  │  │    │  │  CompleteFlowController     │    │ │
│  │  │                       │  │    │  │                             │    │ │
│  │  │  - flows[]            │  │    │  │  - flows[]                  │    │ │
│  │  │  - flow_context       │  │    │  │  - flow_context             │    │ │
│  │  │  - transactional      │  │    │  │  - transactional            │    │ │
│  │  └───────────┬───────────┘  │    │  └──────────────┬──────────────┘    │ │
│  │              │              │    │                 │                   │ │
│  │              ▼              │    │                 ▼                   │ │
│  │  ┌───────────────────────┐  │    │  ┌─────────────────────────────┐    │ │
│  │  │ SimpleFlowSubscriber  │  │    │  │  CompleteFlowSubscriber     │    │ │
│  │  │                       │  │    │  │                             │    │ │
│  │  │  + execute(context)   │  │    │  │  + can_execute?(context)    │    │ │
│  │  └───────────┬───────────┘       │  │  + valid?(context)          │    │ │
│  │              │              │    │  │  + prepare(context)         │    │ │
│  │              │              │    │  │  + save(context)            │    │ │
│  │              │ extends      │    │  │  + dispose(context)         │    │ │
│  │              ▼              │    │  └─────────────────────────────┘    │ │
│  │  ┌───────────────────────┐  │    │                                     │ │
│  │  │SimpleCatchFlowSubscrib│  │    │                                     │ │
│  │  │                       │  │    │                                     │ │
│  │  │  + execute(context)   │  │    │                                     │ │
│  │  │  + catch(err, context)│  │    │                                     │ │
│  │  └───────────────────────┘  │    │                                     │ │
│  └─────────────────────────────┘    └─────────────────────────────────────┘ │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Communication Flow - Simple Flow

**SimpleFlow** executes each subscriber sequentially. Ideal for simple flows where each step depends on the previous one.

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                        SimpleFlowController                                  │
│                      ┌─────────┘                                             │
│                      ▼                                                       │
│  ┌───────────────────────────────────────────────────────────────────────┐   │
│  │  LOOP: For each flow in @flows                                        │   │
│  │                                                                       │   │
│  │    ┌─────────────────┐     ┌─────────────────┐     ┌──────────────┐   │   │
│  │    │  Subscriber A   │ ──► │  Subscriber B   │ ──► │ Subscriber C │   │   │
│  │    │       │         │     │       │         │     │      │       │   │   │
│  │    │       ▼         │     │       ▼         │     │      ▼       │   │   │
│  │    │ execute(context)│     │ execute(context)│     │execute(ctx)  │   │   │
│  │    └─────────────────┘     └─────────────────┘     └──────────────┘   │   │
│  │                                                                       │   │
│  │    flow_context is passed and modified at each step                   │   │
│  │    ──────────────────────────────────────────────────────────────►    │   │
│  └───────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
│  Returns: flow_context (with all modifications)                              │
└──────────────────────────────────────────────────────────────────────────────┘
```

**SimpleCatchFlowSubscriber** adds exception handling:

```
┌─────────────────────────────────────────────────────────────┐
│              SimpleCatchFlowSubscriber                      │
│                                                             │
│   run(flow_context)                                         │
│         │                                                   │

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gmascb) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
