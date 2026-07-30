---
trigger: always_on
description: Micro-frontend architecture: Boundary design. Load when this topic is in scope; part of mfe-skills.
---


# Boundary design

**Version**: 1.2 | **Skill**: understanding-mfe-architecture | **Source**: *Building Micro-Frontends* (O'Reilly) and the Micro-Frontend Canvas

---

## The canonical definition

> A micro-frontend represents a business subdomain that is autonomous, independently deliverable, with the same or different technology, with a low degree of coupling, and owned by a single team.

Six characteristics follow directly from this definition:

- **Business domain representation** — models a subdomain, not a technical layer or UI component type
- **Autonomous codebase** — built and maintained without coordinating with other teams
- **Independent deployment** — shipped without requiring other teams to act
- **Low coupling** — minimal external dependencies; changes inside do not cascade outside
- **Optimised for fast flow** — team can iterate and release at its own cadence
- **Single-team ownership** — one team owns design, development, deployment, and operations end-to-end

**The critical distinction from components**: a component addresses a technical challenge through abstraction and reusability. A micro-frontend completely owns a business domain and is not designed for reuse. If you can reuse it everywhere, it is probably a component. If it represents a domain and can live on its own, it is a micro-frontend.

---

## The seven principles

Adapted from microservices architecture. When reviewing or generating architecture, check it against each one.

**1. Modelled around business domains** — identify boundaries by user journey steps (checkout, search, profile) or business capabilities, not by technical framework types or component categories. DDD event storming is the recommended technique for identifying subdomains.

**2. Culture of automation** — a poor automation culture makes micro-frontends a nightmare. Every independent unit needs solid CI/CD pipelines with a fast feedback loop. Automation also enforces guardrails: bundle size limits, dependency alignment, design system version enforcement, and architecture fitness functions (for example no cross-MFE imports except explicit shared allowlists).

**3. Hide implementation details** — define the API contract upfront. Internal frameworks, data fetching strategies, and code structure are hidden behind it. Teams change their internals freely without affecting others, as long as the contract is respected. This is API-first applied to the frontend.

**4. Decentralise governance** — teams make technical decisions within their domain. Tech leadership provides guardrails; teams operate within them without waiting for central decisions. Avoid one-size-fits-all approaches.

**5. Deploy independently** — a micro-frontend ships without coordinating with other teams. Requiring coordination to release is the primary signal that boundaries are wrong.

**6. Isolate failure** — runtime composition means network failures and 404s are inevitable. Provide graceful fallbacks. A failure in one micro-frontend must not cascade to the shell or other micro-frontends.

**7. Be highly observable** — distributed frontends require logging, monitoring, and the ability to trace a user journey end-to-end. Observability is not optional.

---

## Organisational readiness gate

Apply before any implementation discussion. If the user cannot answer these clearly, the boundary design is premature — recommend a discovery phase first.

**When micro-frontends make sense:**
- Multiple teams need to release independently
- The frontend is a coordination bottleneck — merge conflicts, slow release cycles, communication overhead
- Clear business domains exist that can map to team ownership
- The organisation can invest in platform infrastructure and automation

**When micro-frontends add cost without benefit:**
- Fewer than 2–3 teams need to release independently
- No dedicated platform or infrastructure team — MFEs require governance, shared shell management, and observability infrastructure; without this investment you create chaos instead of autonomy
- Automation pipelines are immature — MFEs amplify deployment problems, they do not solve them
- Short-lived projects or MVPs where the monolith overhead is manageable

When any blocker applies, say so explicitly and recommend the simpler architecture.

---

## Identifying boundaries with domain-driven design

The recommended technique for identifying micro-frontend boundaries is domain-driven design (DDD) applied to the frontend. DDD starts from the assumption that software should reflect what the organisation does — boundaries follow domains, not technical layers.

**Event storming** is the recommended workshop for discovering domains. It brings together people from different backgrounds — product managers, testers, engineers — to build a timeline of the system from a business perspective. The output is a map of subdomains: distinct areas with their own responsibilities, business logic, and vocabulary.

DDD identifies three subdomain types, each requiring different investment:

- **Core subdomains**: the main reasons the application exists. Invest heavily here — developer seniority, code quality, fast feedback loops. Example: the video catalog for a streaming platform.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lucamezzalira/mfe-skills](https://github.com/lucamezzalira/mfe-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
