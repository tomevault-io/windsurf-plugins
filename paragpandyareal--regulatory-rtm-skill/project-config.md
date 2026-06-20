---
trigger: always_on
description: Create comprehensive Regulatory Requirements Traceability Matrices (RTM) from regulatory documents, legislation, rule changes, policies, or compliance standards. Use this skill whenever the user asks to analyse a regulation, extract requirements from a regulatory document, create a traceability matrix, map regulatory obligations, build a compliance matrix, or trace requirements from any official document like legislation, rules, standards, or policy papers. Also trigger when the user uploads a P
---


# Regulatory Requirements Traceability Matrix (RTM) Skill

This skill analyses regulatory documents (legislation, rule changes, standards, policies) and produces a comprehensive Requirements Traceability Matrix that maps every obligation to its source, explains it in plain English, categorises it by business domain, and clearly flags what is verbatim from the source vs what has been derived or assumed.

## When to Use This Skill

Use this whenever the user provides a regulatory document and wants to understand what it requires. This includes new regulations, rule changes, amendments, compliance standards, policy papers, or any official document that creates obligations.

## Core Principle: Honesty About Sources

The most important principle in this skill is **never making things up and always being transparent about where information came from**. Every single entry in the RTM must be clearly marked as one of the following:

| Source Type | What It Means | When to Use |
|---|---|---|
| **VERBATIM** | Exact text copied directly from the regulatory document, word for word | When quoting the actual clause, rule, or requirement text |
| **DERIVED** | A requirement that is not stated word-for-word but is clearly implied or logically follows from the verbatim text | When the regulation says "participants must submit data monthly" and you derive that they need a data submission system |
| **ASSUMED** | Something that would typically be needed to comply but is not explicitly stated in the document | When you flag that training would likely be needed even though the regulation doesn't mention it. Always explain why this assumption was made |
| **INTERPRETED** | Where ambiguous regulatory language has been interpreted in a specific way | When a clause uses vague terms like "reasonable" or "appropriate" and you've interpreted what that means in practice |

If the user hasn't provided enough information to fill a field, leave it as "To be confirmed" rather than guessing.

## RTM Structure

Every requirement extracted from a regulatory document should be captured with the following fields. Read `references/rtm-fields.md` for detailed descriptions and examples of each field.

### Required Fields (always include these)

1. **Requirement ID** - Unique identifier (e.g., REQ-001, REQ-002)
2. **Source Reference** - Exact clause, section, rule, or schedule reference from the source document (e.g., "Schedule 1, Rule 3.2", "Section 14(2)(b)", "Clause 7.1.1")
3. **Source Document** - Name and version/date of the regulatory document
4. **Verbatim Text** - The exact text from the regulation, copied word for word. No paraphrasing, no shortening. This is the single source of truth
5. **Plain English Summary** - What this requirement actually means in simple, jargon-free language that a non-specialist could understand. Write as if explaining to someone who has never read the regulation
6. **Source Type** - One of: VERBATIM, DERIVED, ASSUMED, INTERPRETED (see table above)
7. **Business Domain** - Which area of the business this applies to (e.g., Billing, Metering, IT Systems, Customer Communications, Market Operations, Settlement, Registration, Compliance, Legal, Finance, Network Operations, Data Management)
8. **Obligation Type** - What kind of obligation this creates:
   - **MUST DO** - Mandatory requirement, no choice
   - **MUST NOT DO** - Explicitly prohibited
   - **MAY DO** - Permissive, optional
   - **CONDITIONAL** - Only applies if certain conditions are met
   - **REPORTING** - Requires reporting or disclosure
   - **RECORD KEEPING** - Requires maintaining records
   - **TIMELINE** - Specifies a deadline or timeframe
   - **ENTITLEMENT** - A right granted to one party that creates a corresponding obligation on another party (e.g., "customers are entitled to receive a bill" creates an obligation on the retailer to provide one)
9. **Priority** - Based on obligation type and consequences:
   - **CRITICAL** - Non-compliance has severe consequences (penalties, licence risk, safety)
   - **HIGH** - Mandatory requirement with significant impact
   - **MEDIUM** - Important but with less severe consequences
   - **LOW** - Minor or permissive requirements
10. **Compliance Status** - Current state (defaults to NOT ASSESSED for new analysis):
    - NOT ASSESSED, COMPLIANT, PARTIALLY COMPLIANT, NON-COMPLIANT, NOT APPLICABLE, IN PROGRESS
11. **Effective Date** - When this requirement comes into effect (if specified in the document)

### Recommended Fields (include when possible)

12. **Responsible Party** - Who within the organisation is likely responsible (if determinable from the regulation, e.g., "Registered Participants", "Retailers", "Network Service Providers")

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [paragpandyareal/regulatory-rtm-skill](https://github.com/paragpandyareal/regulatory-rtm-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
