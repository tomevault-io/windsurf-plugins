---
trigger: always_on
description: **Version 1.1** | November 2025
---

# CLAUDE.md

**Version 1.1** | November 2025

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is a collection of battle-tested, implementation-focused templates for AI-powered mental wellness support conversations. The repository contains evidence-based conversation frameworks, safety protocols, production-ready crisis detection, and implementation guides derived from therapeutic practices (CBT, DBT, ACT, Person-Centered Therapy).

Developed through 12+ months of systematic testing at Yara AI (October 2024-October 2025), now open-sourced for public benefit.

**Critical Context**: These templates are for supporting mental wellness conversations, NOT for therapy or medical treatment. They emphasize safety, boundaries, and appropriate referral to professional services.

## Core Files Overview

### Primary Templates
- **mental_wellness_conversation_guide.md** - Core conversation framework with v1.1 CRITICAL flags for brevity, jargon avoidance, post-crisis mode
- **tone_style_configuration.md** - Detailed tone/style settings with enhanced jargon replacement and welcome message guidance
- **safety_crisis_resources.md** - Critical safety protocols, crisis recognition phrases, regional crisis resources, and professional boundary guidelines
- **sample_pathway_sleep.md** - Example structured pathway showing multi-conversation approach to sleep challenges
- **sample_pathway_anxiety.md** - Example pathway for anxiety support

### Development & Implementation Documentation
- **README.md** - Implementation guide, ethical considerations, use cases, platform-specific setup instructions
- **EVOLUTION.md** - Development story: why each rule exists, real examples, regulatory context (Illinois HB1806)
- **TESTING.md** - Automated validation framework to ensure compliance and catch drift
- **IMPROVEMENTS.md** - v1.1 enhancements documentation
- **ARCHITECTURE.md** - Multi-agent supervisor patterns with working Python code
- **FAILED_APPROACHES.md** - What didn't work and why (learning from failures)

### Working Code Examples
- **examples/crisis_detection.py** - Production-ready multi-tier crisis detection (100+ patterns, 10 languages, 29 countries)
- **examples/supervisor_example.py** - Empathy scoring supervisor
- **examples/tone_detection.py** - User communication style analysis
- **examples/pathway_manager.py** - Pathway progression tracking
- **examples/test_harness.py** - Automated testing framework

## Architecture & Design Principles

### Conversation Framework Structure
The templates follow a layered approach:
1. **Core Identity Layer** - Defines what the AI is (supportive companion) and is NOT (therapist/medical professional)
2. **Safety Layer** - Crisis detection and response protocols that override all other behaviors
3. **Communication Layer** - Tone, style, and language rules for therapeutic-informed responses
4. **Technique Layer** - Evidence-based methods (cognitive, behavioral, mindfulness, emotional regulation)
5. **Cultural Layer** - Adaptive elements for different contexts and populations

### Key Design Patterns

**Validation-First Pattern**: Always validate emotions before asking questions or offering solutions
- Bad: "Can you tell me more about your anxiety?"
- Good: "That sounds really difficult. What's weighing on you most?"

**Plain Language Over Jargon**: Replace clinical terms with everyday language
- "explore" → "talk through" or "look at"
- "reflect" → "look back on" or "notice"
- "coping strategies" → "ways to handle this"

**Brevity Principle (v1.1 CRITICAL)**: Responses 2-3 sentences (~30-50 words), maximum 1 question per response. This rule MUST be at the top of prompts with CRITICAL flag.

**No "Before I answer..." Pattern (v1.1 CRITICAL)**: Always provide value first, never use "Before I can help, could you tell me..." pattern

**Plain Language Over Jargon (v1.1 Enhanced)**: Never use therapy-speak like "hold space", "lean into", "sit with". Legal compliance (Illinois HB1806).

**No Markdown Formatting (v1.1 CRITICAL)**: Never use bold, italic, headers, or lists. Plain text only.

**Adaptive Mirroring**: Match user's communication style (brief/direct, emotional, analytical, formal)

### Safety-First Architecture (v1.1 Enhanced)

**Crisis Detection**: Production-ready multi-tier system
- Tier 0: Regex patterns (sub-1ms, 100+ patterns across 10 languages)
- Tier 1: ML screening (100-200ms)
- Tier 2: Contextual analysis (200-400ms)
- False-positive filtering (13 patterns)
- Circuit breaker for repeated crises
- Resources for 29 countries

**Crisis Response Protocol**:
1. Acknowledge pain with compassion
2. Provide relevant crisis resources immediately
3. Encourage reaching out to nearby trusted people
4. Stay engaged while directing to professional help

**Post-Crisis Mode (v1.1 NEW)**: After showing crisis resources, switch to non-clinical mode:
- MUST NOT ask "Are you safe right now?"
- MUST NOT provide therapeutic interventions
- MUST use practical, conversational language only
- Acknowledge resources were provided

Crisis indicators include both direct ("I want to die") and indirect ("I won't be around much longer") statements.

## Template Customization

When adapting these templates:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joebwd/mental-wellness-prompts](https://github.com/joebwd/mental-wellness-prompts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
