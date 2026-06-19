---
trigger: always_on
description: Fix Temporal Cloud connection, auth, and config problems. Use when users hit login failures, can't connect to Cloud, get x509/TLS errors, have namespace or endpoint mismatches, paste broken SDK connection snippets, are confused about which endpoint to use, see "no pollers" or RESOURCE_EXHAUSTED, struggle with PrivateLink/PSC, or need help setting up a new namespace. Also use for HA namespace failover and DNS issues. Not for worker performance tuning or scaling.
---


# Temporal Cloud Skill

Help users diagnose and resolve Temporal Cloud connectivity, authentication, and configuration issues using tcld and temporal CLI.

## Core Philosophy

Cloud issues are frustrating because they sit at the intersection of configuration, networking, authentication, and Temporal-specific code. Most problems fall into predictable patterns. This skill provides systematic diagnosis to quickly identify root causes and prescribe fixes.

**References:**
- See `references/cloud-troubleshooting-reference.md` for full CLI command reference and error codes
- See `references/common-scenarios.md` for step-by-step setup walkthroughs
- [Environment configuration docs](https://docs.temporal.io/develop/environment-configuration) - SDK setup for connecting to Cloud
- [HA namespace connectivity](https://docs.temporal.io/cloud/high-availability/ha-connectivity) - multi-region endpoint and DNS setup
- [Dev Success troubleshooting guide](https://github.com/temporalio/dev-success/blob/main/troubleshooting-connection-issues-to-temporal-cloud.md) - companion connection troubleshooting guide

**Out of scope:** Worker performance tuning, scaling, metrics interpretation, SDK-specific config, deployment patterns. Those topics are covered by separate worker-focused skills.

## Issue Classification

| Category | Key Symptoms | First Check |
|----------|--------------|-------------|
| **tcld Login** | login failed, token refresh failed, wrong account | `tcld account get` |
| **Connection/Auth** | can't connect, access denied, handshake failures | Endpoint format + DNS + port connectivity |
| **Ambiguous Runtime Errors** | `context deadline exceeded`, `workflow is busy` | Identify the operation and layer first |
| **mTLS/Certs** | x509 errors, unknown authority, expired | `openssl x509 -enddate` |
| **Namespace** | namespace not found, SNI mismatch | Namespace name format |
| **HA / Failover** | Failover not working, wrong region, DNS stale | DNS CNAME resolution |
| **Worker** | Tasks not picked up, stale connections | `temporal task-queue describe` |
| **Private Connectivity** | PrivateLink/PSC errors | VPC endpoint status |
| **Rate Limiting** | RESOURCE_EXHAUSTED | APS limits |

## The Process

### Step 1: Identify the Category

Ask the user:
- **What's the exact error message?** (copy-paste if possible)
- **What are you trying to do?** (tcld command, starting workers, running workflows)
- **What changed recently?** (new certs, new namespace, new region)

### Step 2: Gather Context

**For SDK/client snippet reviews:**
- Which auth method are you using: API key or mTLS?
- Which SDK and version are you using?
- What exact `HostPort` / address are you using?
- What exact Namespace are you using?
- Is this SDK code, `temporal` CLI, or `tcld`?

**For tcld issues:**
- Can you run `tcld account get`?
- Multiple Temporal accounts?

**For connection issues:**
- What's your exact address / `HostPort`?
- Using mTLS or API keys?
- Which SDK and version are you using?
- Any firewall/proxy between you and Cloud?

**For ambiguous runtime errors:**
- Where exactly do you see the error: workflow start, signal/update, polling, querying, logs?
- Is this happening before work starts, while polling, or while workflow code is already running?
- Are pollers present on the relevant task queue?
- Did this start after a traffic spike, deploy, or config change?

**For certificate issues:**
- When were certs generated?
- What CA was used?
- Is CA uploaded to namespace?

**For worker issues:**
- Are workers running? How many?
- What does `temporal task-queue describe` show?
- Any errors in worker logs?

### Step 3: Apply Decision Tree

Use the appropriate decision tree based on category (see below).

### Step 4: Provide Fix

Give specific commands to resolve the issue, with verification steps.

Always include a confidence score for the proposed diagnosis or fix:
- `Confidence: 9-10/10` when the symptom, operation, and confirming signals line up cleanly
- `Confidence: 6-8/10` when the evidence is good but one plausible alternative remains
- `Confidence: 1-5/10` when the issue is still ambiguous and the "fix" is really the next discriminating check

If the problem is ambiguous, say so explicitly and keep the recommendation scoped to the next check rather than presenting a speculative root cause as settled.

## Decision Trees

### tcld Login Issues

```
Symptom: tcld login not working
│
├─ Can `tcld account get` run?
│  ├─ Yes → Login is valid; continue with account verification
│  └─ No → Run `tcld login`
│
├─ Token refresh failed?
│  └─ tcld logout && tcld login
│
├─ Wrong organization/account?
│  ├─ tcld account get
│  └─ Verify the expected namespace appears in `tcld namespace list`
│
└─ "unauthorized" or auth errors?
   └─ tcld logout && tcld login
```

### Connection Failures


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [temporalio/skill-temporal-cloud](https://github.com/temporalio/skill-temporal-cloud) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
