---
trigger: always_on
description: >-
---


# Agentic Email Skill

## At a Glance

| What it creates | Best for | Output |
|---|---|---|
| Sales and lifecycle email sequences | Agentic workflow outreach, follow-up, proposal support, and post-sale communication | Branded PDF, Markdown, optional PNG preview |

This skill turns verified buyer, workflow, offer, proof, and CTA details into concise CompleteTech-style email copy. It is local-only: it drafts and renders documents, but it does not send email or call mail-provider APIs.

## Included Email Sets

| Set | Covers |
|---|---|
| Outbound | Cold outreach, warm introductions, first follow-ups, examples, risk-control follow-ups, and breakups. |
| Discovery | Inbound response, qualification, booking, agenda confirmation, and post-discovery recap. |
| Sales motion | Proposal preview, proposal send, objections, close summaries, contract messages, invoice/deposit notes, and kickoff. |
| Post-sale | Delivery updates, review requests, handoff, expansion, referral, testimonial, retention, win-back, and nurture. |

## Use When

Use this skill when verified buyer, workflow, offer, proof, and CTA details need to become polished email copy or a branded PDF email-sequence artifact.

## Boundaries

| This skill does | This skill does not |
|---|---|
| Draft email copy and email sequences. | Send email or call mail-provider APIs. |
| Render branded PDF/Markdown artifacts. | Replace proposal, contract, invoice, delivery, customer-success, or proof artifacts. |
| Use verified facts from other skills as context. | Invent client proof, recipient authority, metrics, legal claims, or regulated-use assurances. |
| Keep messages review-ready and scoped. | Approve external sending or bypass recipient/routing verification. |

## Core Workflow

| Step | Action |
|---|---|
| 1 | Identify the stage: cold outreach, warm intro, follow-up, booked meeting, discovery recap, proposal, close, post-sale, retention, referral, or reactivation. |
| 2 | Gather only the facts needed for that stage. |
| 3 | Use `references/positioning.md` for service promise, risk controls, and language to avoid. |
| 4 | Use `references/email-catalog.md` for template selection, multi-step sequences, or near-exhaustive libraries. |
| 5 | Use `references/sequence-blueprints.md` for outreach and sales cadence design. |
| 6 | Draft in the requested voice, or default to concise, plain, consultative, and specific. |
| 7 | Include subject lines when useful and avoid unsupported claims. |

| Needed Fact | Examples |
|---|---|
| Buyer context | Name, company, role, industry. |
| Trigger | Observed change, operational pain, buying signal, or business problem. |
| Workflow | Agentic workflow being pitched or supported. |
| Message inputs | Proof, constraints, offer, CTA, timing, and tone. |

## Email Selection Guide

Choose by current decision point first, then by buyer persona, then by trigger.

| Decision Point | Templates |
|---|---|
| Cold outbound | `cold-problem-pilot`, `cold-operations-bottleneck`, `cold-technical-evaluation`, `cold-executive-risk`, `cold-revenue-team`, `cold-support-team`, `cold-ops-knowledge`, `cold-founders` |
| Cold follow-up | `followup-workflow-map`, `followup-risk-controls`, `followup-example`, `followup-proofless-value`, `followup-breakthrough-question`, `breakup-close-loop` |
| Warm or inbound | `warm-intro-context`, `warm-intro-workflow`, `inbound-fast-response`, `inbound-qualification`, `inbound-booking` |
| Discovery and proposal | `discovery-confirm-agenda`, `post-discovery-recap`, `proposal-preview`, `proposal-sent`, `proposal-followup-questions` |
| Closing and objections | `close-objection-budget`, `close-objection-risk`, `close-objection-timing`, `close-objection-internal-team`, `close-decision-summary`, `close-final-nudge` |
| Contract and payment | `contract-sent`, `contract-clarifications`, `deposit-invoice`, `signature-reminder` |
| Kickoff and delivery | `kickoff-after-signature`, `kickoff-agenda`, `access-request`, `weekly-update`, `review-ready`, `acceptance-request`, `handoff-complete` |
| Expansion and proof | `expansion-next-workflow`, `referral-request`, `testimonial-request`, `quarterly-checkin`, `winback-new-trigger` |
| Trigger and nurture | `trigger-hiring`, `trigger-new-tool`, `trigger-growth`, `nurture-educational`, `nurture-one-page-offer`, `reengage-old-opportunity` |

| Selection Rule | Guidance |
|---|---|
| Buyer has not agreed to a problem | Stay in cold, follow-up, warm intro, inbound, or discovery. |
| Buyer has agreed to scoped work | Move to proposal, closing, contract, or payment. |
| Project is signed | Use kickoff, delivery, acceptance, handoff, support, or expansion. |
| Several templates fit | Pick the one closest to the buyer's current decision point. |

## Quality Rules

| Rule | Requirement |
|---|---|
| Length | Keep cold emails under 120 words unless the user asks for long-form. |
| Opening | Lead with a concrete operational problem, not generic AI excitement. |
| Positioning | Present agentic development as workflow design, implementation, evaluation, monitoring, and human approval gates. |
| CTA | Use one clear CTA per email. |
| Follow-ups | Add a new angle, artifact, risk reducer, example workflow, or decision prompt. |
| Cold outbound | Include a polite opt-out line when appropriate. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CompleteTech-LLC/agentic-email-skill](https://github.com/CompleteTech-LLC/agentic-email-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
