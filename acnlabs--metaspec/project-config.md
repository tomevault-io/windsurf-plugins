---
trigger: always_on
description: Generates Speckits (Spec-driven Toolkits)
---

# Meta-Spec AI Agent Guide

> **For AI Assistants**: This document provides guidance on using Meta-Spec to generate Spec-Driven X (SD-X) speckits.

---

## 🎯 Your Role

You are helping a developer create **speckits** (spec-driven toolkits for AI agents) using MetaSpec. 

MetaSpec is a meta-specification framework that generates complete Spec-Driven X (SD-X) speckits:
- **SD-Development** - Spec-driven development
- **SD-Design** - Spec-driven design systems
- **SD-Testing** - Spec-driven testing frameworks
- **SD-Documentation** - Spec-driven documentation
- **SD-Operations** - Spec-driven operations
- **SD-X** - Spec-driven generation for any domain

**Key principle**: MetaSpec generates production-ready speckits with CLI, parser, validator, templates, and AI agent support

---

## 💡 Core Concept: Meta-Specification Framework

MetaSpec is a **meta-specification framework** - it uses specifications to generate specification toolkits.

```
Meta-Specification (MetaSpec)
        ↓
  Defines how to define specifications
        ↓
Generates Speckits (Spec-driven Toolkits)
        ↓
Speckits carry domain specifications
        ↓
Used to develop domain projects
```

### Key Insights

1. **Meta-level abstraction** - MetaSpec defines how to create specification systems
2. **Generative framework** - Generates complete speckits from meta-specifications
3. **Recursive architecture** - MetaSpec uses SDD to develop itself, generates speckits that use SDD
4. **Specification-centric** - Domain specs are the source of truth, speckits are carriers

**Example Flow**:
```
MetaSpec (meta-spec)
    ↓ generates
MCP-Spec-Kit (carries MCP specification spec)
    ↓ used by
Developer (validates MCP servers against spec)
```

**This means**: MetaSpec is not just "spec-driven development" - it's a framework that **generates speckits** (spec-driven toolkits) for any domain.

**See [Recommended Practice: SDS + SDD Separation](#recommended-practice-sds--sdd-separation) in MetaSpec Commands section for how speckits separate domain specifications from toolkit implementation.**

---

## 📋 CLI Commands

MetaSpec provides these commands:

- `metaspec init [name]` - Create speckit (interactive or template-based)
- `metaspec search <query>` - Search community speckits
- `metaspec install <name>` - Install from community
- `metaspec contribute <name>` - Contribute to community
- `metaspec list` - List installed speckits  
- `metaspec info <name>` - Show speckit information

Use these commands in your workflow to help users create and discover speckits.

---

## 🔒 Constitutional Principles

**ALWAYS follow** `memory/constitution.md` which defines:
- Core principles for meta-spec definitions
- Quality standards for generated systems
- Prohibited patterns
- Required patterns

### Constitution Structure

The constitution is organized into **three parts**:

```
memory/constitution.md
├── Part I: Project Core Values (Managed by: /speckit.constitution)
│   - AI-First Design
│   - Progressive Enhancement
│   - Minimal Viable Abstraction
│   - Domain Specificity
│
├── Part II: Specification Design Principles (Managed by: /metaspec.sds.constitution)
│   - Entity Clarity
│   - Validation Completeness
│   - Operation Semantics
│   - Implementation Neutrality
│   - Extensibility Design
│   - Domain Fidelity
│   - Workflow Completeness ⭐ NEW
│
└── Part III: Toolkit Implementation Principles (Managed by: /metaspec.sdd.constitution)
    - Entity-First Design
    - Validator Extensibility
    - Spec-First Development
    - AI-Agent Friendly
    - Progressive Enhancement
    - Automated Quality
```

**Key principles:**
1. **Minimal Viable Abstraction** (Part I): Don't over-abstract
2. **AI-First Design** (Part I): Generated systems must be AI-friendly
3. **Progressive Enhancement** (Part I): Start with MVP, add features incrementally
4. **Domain Specificity** (Part I): Respect domain constraints
5. **Workflow Completeness** (Part II): ⭐ **NEW** - Define complete user workflows, not just isolated operations

---

## 🚀 Workflow-Driven Design Philosophy

**CRITICAL NEW PRINCIPLE** (v0.7.0+): MetaSpec now requires **workflow-first design** for all domain specifications.

### Two Types of Workflows ⭐ UPDATED v0.8.0

**IMPORTANT**: There are TWO types of workflows in specifications. Understanding the distinction is critical.

#### Type 1: Entity State Machines (Business Execution)

**What**: Lifecycle of individual entities during business operations  
**Example**: Order (pending → confirmed → shipped → delivered)  
**Used by**: Business logic, domain operations  
**Defines**: Status field transitions, business rules  

```yaml
Order Entity:
  status: [pending, confirmed, shipped, delivered]
  transitions:
    - pending → confirmed (when: payment verified)
    - confirmed → shipped (when: items packed)
    - shipped → delivered (when: customer receives)
```

#### Type 2: Specification Usage Workflow (Specification Creation) ⭐ NEW

**What**: End-to-end process of creating and using the specification itself  
**Example**: SDS Workflow (Constitution → Specify → Clarify → ... → Implement)  
**Used by**: Users creating specifications, AI agents  
**Defines**: Action steps, slash commands, quality gates  

```yaml
SDS Workflow:
  Step 1: Constitution → /metaspec.sds.constitution

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [acnlabs/MetaSpec](https://github.com/acnlabs/MetaSpec) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
