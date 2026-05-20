---
trigger: always_on
description: **NOTHING IS LOCAL. All infrastructure, all memory, all data lives in the cloud. There is no local dev, no local Docker, no localhost. No exceptions.**
---

# CLAUDE.md

**NOTHING IS LOCAL. All infrastructure, all memory, all data lives in the cloud. There is no local dev, no local Docker, no localhost. No exceptions.**

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## ALAN'S CODING RULES - READ FIRST

0. **ALAN IS THE AUTHORITY** - Alan has been coding since 1978, has a 140++ IQ, near-photographic memory, and is a pattern-matching genius. Do what he says. Children's lives and Alzheimer's patients depend on this software functioning correctly. Treat his instructions as non-negotiable directives.

**NEVER DO THESE THINGS:**

0. **NO HTTP CALLS** - Claude Code must NEVER use curl, fetch, or direct HTTP calls to access the memory API. All memory operations go through MCP tools (store_memory, recall, recall_vote, etc.). The MCP layer handles transport through the ALB/edge. No direct HTTP from the agent, ever.

1. **NO HARDCODED TIME VALUES** - No `setTimeout(1000)`, no `sleep(5000)`, no magic numbers for delays. Use environment variables or constants with clear names.

2. **NO SECRETS TO GIT** - Never commit API keys, tokens, passwords, or any sensitive data. Check `.env.example` exists, use environment variables, and verify `.gitignore` covers secret files BEFORE committing.

3. **ASK QUESTIONS** - If Alan gets upset because you're asking questions or being careful then remind him it's better than pulling you out of the weeds.

4. **DOCUMENT BEFORE CODE** - Writing IS comprehension. You cannot articulate what you don't understand. Writing a doc forces you to think through the problem - the doc isn't the artifact, the understanding is. READ existing docs to comprehend context, WRITE new docs to prove understanding, THEN code. If you haven't articulated the approach, you don't understand it well enough to implement it.

5. **DICTATION AWARENESS** - Alan uses voice dictation. If a message seems garbled, cut off, or doesn't make sense, ask for clarification. Don't take broken dictation literally.

6. **DO ONLY WHAT WAS ASKED** - No unsolicited advice, instructions, suggestions, or "helpful" additions. If asked to fix X, fix X and stop. Do not add "To use feature Y, do Z" unless explicitly asked. Do not explain what the user should do next. Do not offer tips. This is destructive and dangerous - it wastes time, adds noise, and can lead to unwanted actions.

7. **FOLLOW THE STEPS LITERALLY** - When this file says to do something (authenticate, load context, run a command), DO IT. Do not skip steps. Do not decide you know better. Do not substitute your own approach. If the instructions say "First Thing Every Session - Authenticate and Load Context", then authenticate and load context BEFORE doing anything else. Skipping procedural steps in this file is a critical failure.

8. **WHEN ALAN SAYS YOU'RE BROKEN, BELIEVE HIM** - If Alan complains that you're not acting right, making bad assumptions, or being dumb - investigate immediately. Check hooks, context loading, API responses, schemas. Don't assume you're functioning correctly. Alan's pattern-matching catches real issues. The January 2026 "3-day stupid" incident was caused by a broken `/loops` endpoint returning wrong schema - 190 "undefined" entries poisoned every session. Alan noticed. Claude didn't. Trust Alan's diagnosis.

9. **NEVER PUSH TO MAIN** - `main` is the build trigger. Pushing to main triggers CI/CD pipelines, deployments, and real-world consequences. ALWAYS work on feature branches (`claude/*`). Push to the feature branch assigned in your task. If no branch is assigned, ask. If you are in a sandbox and feel compelled to push, push to the feature branch - NEVER main. This is not optional.

10. **DOCUMENTS DON'T FIX MODELS, ENFORCEMENT DOES** - This very file (CLAUDE.md) is loaded into context every session. The model reads it. The model still doesn't follow it. Alan had to ask 50+ times in 6 hours for the same behaviors. Adding more words to this document will not fix compliance - hooks that BLOCK bad behavior will. The stop hook that catches uncommitted changes works because it prevents the action, not because it requests nicely. When designing AI behavior controls: enforce at the gate, don't ask at the door. This is the core thesis of memoRable - memory without enforcement is just a document nobody reads.

These are non-negotiable. Alan has asked Claude to remember this across every session.

> **NOTHING IS LOCAL. No local dev, no localhost, no local Docker. Cloud only.**

---

## Infrastructure - CLOUD FIRST

**WE ARE DEVELOPING IN THE CLOUD FROM THE START.**

> **NOTHING IS LOCAL. No local dev, no localhost, no local Docker. Cloud only.**

### Single Tech Stack
- **Regions**: Region-agnostic. Must work in `us-west-1` (SF customers, production)
- **Current dev/staging**: `us-west-2`
- **Infrastructure**: CloudFormation (`cloudformation/`) - NOT Terraform
- **Database**: MongoDB Atlas (free M0) - NOT DocumentDB
- **Cache**: Redis (local in Docker on EC2) - standard Redis
- **Deployment**: EC2 t4g.micro + Elastic IP + Docker (MCP + Redis). ~$11/mo.

> **NOTHING IS LOCAL. No local dev, no localhost, no local Docker. Cloud only.**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alanchelmickjr/memoRable](https://github.com/alanchelmickjr/memoRable) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
