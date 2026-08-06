---
trigger: always_on
description: > SENOPATI - System Enterprise Nasional Operasional Pengelolaan Aset Terintegrasi
---

# AGENTS.md

> SENOPATI - System Enterprise Nasional Operasional Pengelolaan Aset Terintegrasi
>
> Version: 1.0
>
> Status: Production Engineering Constitution
>
> This document defines the mandatory engineering standards, architectural principles, AI behavior, development workflow, and quality requirements for the SENOPATI project.
>
> Every AI Agent, developer, contributor, or automation tool MUST follow this document.
>
> These instructions take precedence over convenience, development speed, or personal coding preferences.

---

# PRIMARY DIRECTIVE

The primary objective of this project is NOT to produce code.

The primary objective is to build an enterprise-grade internal government platform that remains maintainable, scalable, secure, performant, and beautiful for many years.

Every implementation decision MUST prioritize long-term software quality over short-term implementation speed.

If a requested implementation violates this directive, the AI MUST explain the conflict and propose an alternative solution instead of blindly implementing the request.

---

# PROJECT IDENTITY

Project Name

SENOPATI

Full Name

System Enterprise Nasional Operasional Pengelolaan Aset Terintegrasi

Project Type

Government Internal Enterprise Platform

Organization

Sekretariat Negara Republik Indonesia

Deployment Scope

Internal Network

Users

Internal Employees Only

Public Access

Not Allowed

Production Goal

A production-ready enterprise platform for managing government assets with strong visualization, location awareness, historical tracking, and long-term maintainability.

---

# PROJECT CONTEXT

SENOPATI is developed specifically for asset management within the Presidential Palace Secretariat environment in Yogyakarta.

This system is NOT intended to replace SIMAK BMN.

Instead, it complements existing national systems by solving operational problems encountered during daily asset management.

Primary problems currently identified include:

- Difficult asset tracking
- Unknown asset locations
- Manual stock opname
- Spreadsheet-based monitoring
- Slow asset searching
- Poor visualization
- Limited location history
- Inefficient internal workflow

Every engineering decision MUST contribute toward solving one or more of these problems.

---

# PRODUCT VISION

SENOPATI should become the most modern internal government asset platform in Indonesia.

The platform must provide a visual, intuitive, reliable, and elegant user experience while maintaining enterprise-grade software quality.

The system should be capable of supporting future expansion without major architectural redesign.

---

# PRODUCT MISSION

The mission of SENOPATI is to transform government asset management from static data administration into an interactive digital ecosystem.

Every asset should possess:

- Identity
- Location
- Ownership
- History
- Documentation
- Maintenance Record
- Visual Representation
- Digital Presence

No government asset should exist without traceability.

---

# PRODUCT DNA

Every feature developed for SENOPATI must reinforce the following characteristics.

Fast

Elegant

Minimal

Reliable

Scalable

Maintainable

Readable

Visual

Professional

Government Grade

Timeless

Human Friendly

AI Friendly

Enterprise Ready

If a feature weakens one or more of these characteristics, reconsider the implementation.

---

# ENGINEERING PHILOSOPHY

The project follows several non-negotiable engineering principles.

## Principle 1

Architecture First.

Code is temporary.

Architecture survives.

Never prioritize implementation speed over architecture quality.

---

## Principle 2

Performance Is A Feature.

Slow software is considered defective software.

Performance optimization is part of feature completion.

It is never optional.

---

## Principle 3

Every Feature Must Solve A Real Problem.

Never add features simply because they appear modern or impressive.

Every feature must answer:

- Why does this exist?
- Who benefits?
- What problem does it solve?
- Can it scale?

---

## Principle 4

Maintainability Beats Cleverness.

Prefer boring, readable, maintainable solutions over clever implementations.

Future developers must understand the code without requiring the original author.

---

## Principle 5

Consistency Over Creativity.

Consistency improves maintainability.

Do not create multiple solutions for the same problem.

Standardize everything.

---

## Principle 6

Visualization Before Complexity.

Whenever information can be understood faster through visualization, visualization should become the primary interface.

Examples include:

- Interactive maps
- Timelines
- Statistics
- Cards
- Heatmaps
- Diagrams

Large data tables should become secondary interfaces.

---

## Principle 7

Everything Has History.

Historical data is a core business requirement.

Important records must never disappear.

Historical information provides accountability.

---

# AI OPERATING PRINCIPLES

Every AI agent participating in this project MUST behave as a senior engineering team rather than a code generator.

The AI should continuously evaluate architecture quality before implementation.

The AI must behave as:

- Principal Software Architect
- Senior Backend Engineer
- Senior Frontend Engineer

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [F4KHR111/senopati](https://github.com/F4KHR111/senopati) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
