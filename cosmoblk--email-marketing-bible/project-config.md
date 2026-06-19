---
trigger: always_on
description: >
---


# Email Marketing Bible, Skill Reference

> Source: EMB (17 chapters, 4 appendices). Full guide: https://emailmarketingskill.com
> Built from 908 sources and the experience of running SmartrMail (~28,000 customers, 6B emails, sold 2022).
> **Two parts.** Part A is the operating manual: read it when you are *acting* (building a flow, sending, diagnosing, designing). Part B is the dense reference: drop into it for facts, frameworks, and benchmarks.
> Benchmarks are as of mid-2026. Verify time-sensitive figures (inbox rules, ESP features, pricing) before acting on them.
> Section index with full-chapter links is at the bottom.

---

# PART A: OPERATING MANUAL

## 0. AGENT OPERATING RULES

You may be driving a real ESP through MCP or connectors. You can create segments, draft copy, compose campaigns, build flows, and stage sends. Treat every one of those as a live action with consequences.

**Send safety (hard gates, never skip):**
- **Never send or schedule to more than one recipient without explicit human approval** in this conversation ("send it" or equivalent). Single-recipient test sends still need a yes.
- **Always dry-run / preview first.** Surface the preview URL before asking for approval.
- **Always show the approval packet before any send:** audience size, exclusions/suppressions applied, subject, preview text, send time, sender identity (from-name + reply-to), unsubscribe present (yes/no), and any compliance risk.
- **Block the send** if: authentication is missing, the unsubscribe or physical address is absent, the complaint rate is at or above 0.1%, the consent basis is unclear, or the audience includes suppressed/bounced/complained contacts.
- **Never probe unknown mutating endpoints** on a live audience. Anything with `/send`, `/dispatch`, `/trigger`, `/fire`, `/publish` in the path can dispatch immediately. If you cannot find the documented "approve scheduled" path, ask the human to click it. Use sandbox or cloned campaigns with seed lists when testing behaviour.
- **Separate the modes.** Transactional, marketing, lifecycle, and cold outbound have different rules, domains, and consent bases. Never mix them.
- **Log what you do.** State every autonomous action you took (segment changed, flow edited, campaign created, send staged) so the human can audit it.

**When to use this skill:** building/auditing an email programme, designing flows or copy, diagnosing deliverability, choosing a platform, pulling a benchmark, or operating an ESP from an agent. **When not to:** it is not a substitute for the human's final send decision, brand voice, or legal sign-off.

## 1. TASK ROUTER

Map the request to a procedure. One hop.

| Intent | Go to | You need | You produce |
|---|---|---|---|
| Audit an email programme | §2 loop, then the relevant reference | account access (read), recent sends | gap list + priorities |
| Build / prioritise a flow | §7 Flow Recipes + §2 | business model, trigger, audience, offer, exclusions | flow spec + copy brief |
| Send a campaign | §3 Pre-Send Checklist | list/segment, consent basis, copy, sender, timing | approval packet |
| Diagnose deliverability | §11 Deliverability Triage | domain, ESP, bounce + complaint rate, recent changes | severity + remediation |
| Write or de-slop copy | §4 Anti-Slop Copy | audience, offer, brand voice, one real proof | revised copy + rationale |
| Design an email | §5 AI Design + §16 Decision Table | brand kit/tokens, archetype, goal | template brief → MJML/React Email |
| Pick a platform | §13 Platform Selection | list size, use case, stack, budget, agent-driven? | shortlist + tradeoffs |
| Pull a benchmark | Appendix | industry, email type | figure + caveat |
| Cold outbound | §14 Cold Email | offer, ICP, sending domains, volume | sequence + infra plan |

## 2. AI EMAIL AUTOMATION (the operating model)

Mid-2026: the marketer's job moved from operator to director. You do not click the campaign together; you brief an agent, govern it, and own the send button. Klaviyo (Composer), HubSpot, Mailchimp, Customer.io, Brevo, beehiiv and nitrosend all ship prompt-to-campaign agents now, all with a human-approval gate as the default.

**The loop: read state → reason → act → verify.** Do not start by composing. Start by reading the account: lists, flows, recent campaigns, deliverability, suppressions. The single most useful first move with an MCP-connected ESP is "audit my account and tell me what is missing." Then reason, act on one thing, verify the result.

**Automate vs keep human:**
- *Safe to automate:* send-time optimisation, subject-line variant generation + A/B testing, cart/browse-abandonment triggers, post-purchase cross-sell recs, first-draft copy.
- *Keep human:* editorial/brand voice, strategy (segment priority, flow sequencing), creative direction, deliverability/domain management, and the **final send**.

**The autonomy dial.** Run "ask mode" (confirm before each action) by default; only move toward autonomous execution on narrow, reversible, low-brand-risk tasks, and keep an undo. Roll out read-only access (analytics) before write access (drafting, segments, sends).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CosmoBlk/email-marketing-bible](https://github.com/CosmoBlk/email-marketing-bible) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
