---
trigger: always_on
description: Intent reference: service blueprinting methodology, frontstage/backstage mapping, touchpoint analysis, moment-of-truth design, channel orchestration. Load when mapping services, systems, or cross-channel experiences.
---


# Service Design

## Service Blueprinting Methodology

A service blueprint is a diagram that maps the complete service delivery process — what the customer sees, what the organization does behind the scenes, and how the technical systems support it all. Lynn Shostack introduced the concept in 1984 to address a fundamental problem: services are invisible, and you can't improve what you can't see.

### Shostack's Original Model

Shostack's original blueprint was deliberately simple: a horizontal timeline of the service process with a "line of visibility" separating what the customer sees from what they don't. Above the line: the customer's experience. Below the line: the backstage processes that produce that experience.

**Why it mattered:** Before Shostack, services were designed through intuition and anecdote. The blueprint made the invisible visible — showing where the service was fragile, where it depended on specific people, and where it failed predictably.

### Modern Blueprint Layers

Contemporary service blueprints expand Shostack's model to five layers, each providing different analytical leverage.

**Layer 1: Customer Actions**
What the customer does at each stage of the service. Their decisions, interactions, and movements across channels. This is the user journey — but in a service blueprint, it's one layer of a larger picture, not the whole picture.

Document: What the customer is trying to accomplish, what they physically do, and what channels they use. Note emotional states where you have evidence — but be careful about projecting emotions you haven't researched.

**Layer 2: Frontstage (Onstage) Actions**
What the customer-facing parts of the organization do that the customer can see. The website, the app, the retail store, the call center agent, the email. This is the visible surface of the service.

Document: Every touchpoint the customer interacts with, and every employee action the customer witnesses. The frontstage is where brand perception is formed — it's also where most organizations focus their design effort, sometimes to the exclusion of everything below.

**Layer 3: Backstage Actions**
What the organization does that the customer can't see but that directly supports the frontstage. The warehouse worker who picks the order. The algorithm that generates recommendations. The support agent who reviews a flagged transaction. The designer who creates the email template.

Document: Every internal action that supports a frontstage interaction. This is where efficiency, consistency, and quality are determined — and where most service failures originate.

**Layer 4: Support Processes**
The systems, tools, and organizational processes that enable backstage actions. CRM systems, inventory management, payment processing, identity verification, internal communication tools. These are typically owned by different teams, different departments, sometimes different companies.

Document: The technical systems and organizational processes that backstage actors depend on. This is where cross-functional dependencies live — and where improving one team's process can break another team's workflow.

**Layer 5: Physical Evidence**
The tangible artifacts the customer encounters at each stage. The packaging, the receipt, the app notification, the confirmation email, the physical product itself. Physical evidence shapes expectations and creates memories.

Document: Every artifact the customer receives, sees, or keeps. Physical evidence persists after the interaction ends — it's what the customer takes away.

### The Lines

Service blueprints are divided by three horizontal lines:

**Line of Interaction** — Between Customer Actions and Frontstage. Where the customer and the service provider interact directly. Every crossing of this line is a moment of truth.

**Line of Visibility** — Between Frontstage and Backstage. What the customer can see vs. what they can't. The strategic question at this line: should we make more of the backstage visible (building trust through transparency) or keep it hidden (reducing complexity)?

**Line of Internal Interaction** — Between Backstage and Support Processes. Where human backstage actors interact with technical systems and organizational processes. This line reveals technology dependencies and process bottlenecks.

---

## Moment-of-Truth Analysis

Not all touchpoints are equal. Moments of truth (a term coined by Jan Carlzon, then CEO of SAS Airlines, in 1987) are the critical interactions that disproportionately shape the customer's overall perception of the service.

### Identifying Moments of Truth

A moment of truth has three characteristics:
1. **High emotional stakes** — The customer cares about the outcome. The anxiety of a medical test result, the anticipation of a delivery, the frustration of a failed payment.
2. **Perception formation** — The interaction shapes how the customer perceives the entire service. A smooth onboarding creates a halo effect. A botched recovery from an error colors everything.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ghaida/intent](https://github.com/ghaida/intent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
