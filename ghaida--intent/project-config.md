---
trigger: always_on
description: Adapts experiences across cultures and languages — not just translation, but cultural reconception. Part of the Intent design strategy system. When a product enters a new market, everything is in play: information density, navigation patterns, color meaning, icon comprehension, date formats, trust signals, payment flows, and the fundamental assumptions about how people make decisions. Trigger when: planning international expansion, auditing i18n readiness, adapting designs for RTL languages, rev
---



# Localize — Design Across Cultures

## Overview

Localization is not translation. Translation converts words; localization adapts the experience.

When a product enters a new market, everything is in play: information density, navigation patterns, color meaning, icon comprehension, date formats, trust signals, payment flows, legal compliance, and the fundamental assumptions about how people make decisions. A checkout flow designed for US consumers doesn't become a Japanese experience by translating the strings. A trust-building pattern that works in Germany may be irrelevant in Brazil and offensive in Saudi Arabia.

Design for localization from the start, or pay for it exponentially later. Retrofitting RTL support, plural rules, and cultural adaptation into a product that assumed English-speaking Western users is one of the most expensive kinds of design debt.

**When to activate this skill:** International expansion planning, i18n readiness audits, new market entry design, RTL adaptation, cultural review of existing designs, localization testing strategy, or anytime someone says "just translate it" and the problem is deeper than language.

---

## Skill family

Localize works alongside the full Intent skill system:

- **`/articulate`**: Everything they write will be localized. Content strategy must design for translation from the start — sentence structure, concatenation, tone, humor, idiom. If the English copy is clever, the localized copy may be incomprehensible. Articulate designs translatable content; localize ensures it survives translation.
- **`/organize`**: Navigation and labeling may need cultural adaptation. Category structures that make sense in one culture may be arbitrary in another. Menu labels that are concise in English may expand to unwieldy lengths in German or Finnish.
- **`/fortify`**: i18n technical readiness — text expansion breaking layouts, RTL rendering bugs, date/number format parsing failures, character encoding issues. Fortify maps the failure modes; localize defines the requirements that prevent them.
- **`/strategize`**: Market analysis and audience definition per locale. Which markets, in what order, with what level of adaptation? `/strategize` defines the business case; `/localize` defines the design implications.
- **`/investigate`**: Cultural research methods for unfamiliar markets. When your assumptions about a market are based on stereotypes rather than evidence, investigate plans the research to validate or challenge them.
- **`/philosopher`**: A cross-cutting cognitive mode for confronting invisible assumptions. Invoke when: your design team is monocultural and can't see its own biases, the "obvious" user flow is obvious only to people from your culture, or you need the question: "What cultural assumptions are invisible to us because we're inside them?"

---

## Core capabilities

### 1. Cultural dimension analysis for UX

Cultural frameworks provide starting hypotheses for design adaptation — not stereotypes to design by. Use them to generate questions, then validate with research.

**High-context vs. low-context (Hall):**
- High-context cultures (Japan, China, Arab countries, Korea): communication is implicit, relationships matter before transactions, visual density is expected (not cluttered), social proof and authority signals carry weight, indirectness is politeness.
- Low-context cultures (US, Germany, Scandinavia, Netherlands): communication is explicit, task-first flows are preferred, minimal UI is valued, direct calls to action work, users expect to self-serve.
- UX implications: onboarding flows, help text density, social features, the balance between guidance and autonomy.

**Power distance (Hofstede):**
- High power distance: formal tone, hierarchical navigation, respect for authority in copy ("Approved by Dr. [Name]"), institutional trust signals.
- Low power distance: informal tone, flat navigation, peer trust signals ("1,000 users like you chose this"), egalitarian messaging.
- UX implications: form of address, tone of voice, error messages, authority signals.

**Individualism vs. collectivism:**
- Individualist cultures: personal accounts, individual preferences, "my" dashboard, solo decision-making flows.
- Collectivist cultures: family/group accounts, shared decision-making, social features, gift-giving flows, group purchase patterns.
- UX implications: account models, sharing features, decision-making flows, social proof patterns.

**Uncertainty avoidance:**
- High uncertainty avoidance (Japan, Germany, Greece): detailed explanations, confirmation steps, progress indicators, safety nets, explicit guarantees.
- Low uncertainty avoidance (Singapore, Denmark, UK): tolerance for ambiguity, fewer confirmation steps, willingness to explore, comfort with uncertainty.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ghaida/intent](https://github.com/ghaida/intent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
