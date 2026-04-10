---
trigger: always_on
description: >-
---


# CRO Funnel Optimization

> **Source:** This skill is adapted from **[AgentKits Marketing](https://github.com/aitytech/agentkits-marketing)** 
> by AITYTech. Enterprise-grade AI marketing automation (MIT License).

Full-funnel Conversion Rate Optimization covering every stage from page visit to paid conversion:

```
Visitor → Page CRO → Form CRO → Signup CRO
     ↓
  Popup CRO (capture abandoners)
     ↓
New User → Onboarding CRO → Activation
     ↓
Free User → Paywall CRO → Paid Customer
```

---

## 1. Form CRO

Optimize lead capture forms, contact forms, demo requests, and quote forms (not signup/registration).

### Core Principles

1. **Every Field Has a Cost** - Each field reduces completion by 10-25%
2. **Value Must Exceed Effort** - Clear value proposition above form
3. **Reduce Cognitive Load** - One question per field, logical grouping

### The 5-Field Rule

| Fields | Typical Impact |
|--------|---------------|
| 3 fields | Baseline |
| 4-6 fields | 10-25% reduction |
| 7+ fields | 25-50%+ reduction |

For each field, ask:
- Is this absolutely necessary before we can help them?
- Can we get this information another way?
- Can we ask this later?

### Field Optimization

**Email**: Single field, inline validation, typo detection

**Name**: Test single "Name" vs. First/Last split

**Phone**: Make optional if possible, explain why if required

**Message**: Make optional, expand on focus

### Multi-Step Forms

Use when:
- More than 5-6 fields needed
- Logically distinct sections
- Conditional paths based on answers

Best practices:
- Progress indicator (step X of Y)
- Start with easy, end with sensitive
- Allow back navigation
- Save progress

### Submit Button Copy

**Weak**: "Submit", "Send"

**Strong**: "[Action] + [What they get]"
- "Get My Free Quote"
- "Download the Guide"
- "Request Demo"

---

## 2. Signup Flow CRO

Optimize registration, account creation, and trial activation flows.

### Core Principles

1. **Minimize Required Fields** - Email + Password minimum
2. **Show Value Before Asking** - Can they experience product first?
3. **Reduce Perceived Effort** - "Takes 30 seconds"

### Field Priority

| Priority | Fields |
|----------|--------|
| Essential | Email, Password |
| Often needed | Name |
| Usually deferrable | Company, Role, Team size, Phone |

### Social Auth Options

- B2C: Google, Apple, Facebook
- B2B: Google, Microsoft, SSO
- Consider "Sign up with Google" as primary

### Single-Step vs. Multi-Step

**Single-step works**: 3 or fewer fields, simple B2C, high-intent visitors

**Multi-step works**: More than 3-4 fields, B2B needing segmentation

**Progressive commitment pattern**:
1. Email only (lowest barrier)
2. Password + name
3. Customization questions (optional)

### Trust Elements

- "No credit card required" (if true)
- "Free forever" or "14-day free trial"
- Privacy note: "We'll never share your email"
- Testimonial near signup form

---

## 3. Onboarding CRO

Optimize post-signup user activation and time-to-value.

### Core Principles

1. **Time-to-Value Is Everything** - How quickly can someone experience core value?
2. **One Goal Per Session** - Don't teach everything at once
3. **Do, Don't Show** - Interactive > Tutorial
4. **Progress Creates Motivation** - Show advancement, celebrate completions

### Define Your Aha Moment

The action that correlates most strongly with retention:

| Product Type | Aha Moment |
|-------------|------------|
| Project management | Create first project + add team member |
| Analytics | Install tracking + see first report |
| Design tool | Create first design + export/share |
| Collaboration | Invite first teammate |
| Marketplace | Complete first transaction |

### Onboarding Checklist Pattern

Best practices:
- 3-7 items (not overwhelming)
- Order by value (most impactful first)
- Start with quick wins
- Progress bar/completion %
- Celebration on completion

Example item:
```
☐ Connect your first data source (2 min)
  Get real-time insights from your existing tools
  [Connect Now]
```

### Empty States

Empty states are onboarding opportunities:
- Explain what this area is for
- Show what it looks like with data
- Clear primary action to add first item
- Optional: Pre-populate with example data

### Handling Stalled Users

1. **Detection**: Define "stalled" criteria (X days inactive)
2. **Email sequence**: Reminder of value, address blockers, offer help
3. **In-app recovery**: Welcome back message, pick up where left off
4. **Human touch**: Personal outreach for high-value accounts

---

## 4. Popup CRO

Optimize popups, modals, overlays, slide-ins, and banners.

### Core Principles

1. **Timing Is Everything** - Too early = annoying, too late = missed opportunity
2. **Value Must Be Obvious** - Clear, immediate benefit
3. **Respect the User** - Easy to dismiss, remember preferences

### Trigger Strategies

| Trigger | When to Use | Example |
|---------|-------------|---------|
| Scroll-based (25-50%) | Blog posts, long-form content | "You're halfway through—get more" |
| Exit intent | E-commerce, lead gen | "Wait! Before you go..." |
| Click-triggered | Lead magnets, demos | Zero annoyance |
| Time-based (30-60s) | Proven engagement | General site visitors |

### Popup Types

**Email Capture**: Newsletter subscription
- Clear value prop (not just "Subscribe")

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dirnbauer) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
