---
trigger: always_on
description: > This file orients any AI agent (and human) working in this repository. Read it fully before making changes. It describes **what Blue Spec is**, **who it is for**, **how it is built**, and **how to work in this codebase**. For the file/folder layout and concrete structure, invoke the internal `/architecture` skill.
---

# Blue Spec: Security-Driven Hardening (SDH)

> This file orients any AI agent (and human) working in this repository. Read it fully before making changes. It describes **what Blue Spec is**, **who it is for**, **how it is built**, and **how to work in this codebase**. For the file/folder layout and concrete structure, invoke the internal `/architecture` skill.

---

## 1. What this workspace is

This workspace is the **development environment for Blue Spec**.

Blue Spec is the practice of **Security-Driven Hardening (SDH)**: a structured, AI-driven security workflow. **Blue Spec turns a codebase into a more secure one**, driving the work from a spec the agent runs rather than from ad-hoc fixes.

The name is inspired by **Blue Teams**: the defenders in security. Blue Spec is about defense, hardening, and verification, never offense.

### The problem it exists to solve

Software is increasingly shipped by people, and AI agents, who cannot reliably recognize an insecure pattern: a leaked secret, a missing authorization check, an injectable query. AI coding assistants made this acute by letting non-developers ship real software, but the gap is broader than any one audience. Most projects never get a security pass that is specific to what they actually do. The ecosystem becomes more fragile with every "it works, ship it" moment.

Blue Spec's mission is to **put a defensive, security-first workflow within reach of any user**, by making the AI agent do the heavy lifting: detecting what a system actually is, and guiding it toward the security practices that matter for that system, in a safe-by-default way.

### What Blue Spec actually ships

Blue Spec is **a collection of templates and agent commands**, not a heavy framework. Its value is the **workflow the AI agent runs**: detecting what the system actually does (login, uploads, payments, and so on), mapping the vulnerabilities that matter for that context, then proposing, applying, and validating the right fixes. The context detection is what makes the rest specific instead of generic.

Concretely, the user runs Blue Spec via the AI agent (today, Claude Code) and the agent follows Blue Spec's templated, security-first process: scaffolded templates plus slash commands that an AI agent consumes, oriented toward defense rather than feature delivery.

### Flexible by design

Blue Spec is meant to flex, never to impose a rigid contract. Its principles, requirements, and recommendations are starting points the user can adapt, not a fixed contract. The charter can grow or shrink, a principle can be reworded or dropped, and every phase adapts to what the project actually is rather than forcing the project to fit a template. Safe-by-default never means take-it-or-leave-it.

---

## 2. Who it is for

Blue Spec is for **any user who wants a development flow with the security practices Blue Spec specializes in**. "User" means a person or a system, developer or not, and **all are served the same way**. There is no primary or secondary audience. The project must be **self-sufficient**, so the experience does not depend on the user already knowing what to ask for.

What makes that possible is the core idea: Blue Spec is **not a static catalog of generic recommendations**. It is a spec **intelligent enough to detect the context of the system** it is run against, for example whether the project has login, file uploads, payments, and so on. From that detection it **directs the user to the specific recommendations that matter for that context**.

This is why developers and non-developers are served equally: the intelligence lives in the spec, not in the user. Findings and recommendations should still be expressed in plain language, so they are actionable regardless of the user's technical depth.

---

## 3. Technology stack

- **Development language: TypeScript.** All source is authored in strict TypeScript.
- **Execution / distribution language: JavaScript.** End users never run TypeScript. The project is **built/compiled to JavaScript** and that build is what ships.
- **Distribution channel: npm, run via `npx`.** The intended end-user entry point is something like `npx blue-spec ...`. This is the friendliest path for the AI / vibe-coder audience and the native idiom of the JS/TS ecosystem.

---

## 4. The Blue Spec workflow (phase model)

Blue Spec runs a structured lifecycle with every phase framed around **defense**. This is the **5-phase Blue Team flow** and it is the product's spine:

| #   | Command             | Blue Team purpose                                                                                               |
| --- | ------------------- | --------------------------------------------------------------------------------------------------------------- |
| 1   | `/bluespec.charter` | Establish the project's **security principles** (the rules)                                                     |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wellwelwel/blue-spec](https://github.com/wellwelwel/blue-spec) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->
