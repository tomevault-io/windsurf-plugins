---
trigger: always_on
description: Write documentation for various platform features using Diataxis framework and quality principles.
---


# Writing Documentation

## Content Classification (Diataxis Framework)

Before writing, identify the content type to determine the appropriate structure.

### The Four Content Types

Diataxis organizes documentation into four types based on two axes:
- **Horizontal axis**: Learning-oriented (left) vs Information-oriented (right)
- **Vertical axis**: Doing/Practical (top) vs Information/Theoretical (bottom)

| Type | User Goal | Orientation | Characteristics | Examples |
|------|-----------|-------------|-----------------|----------|
| **TUTORIAL** | Learn by doing | Learning + Practical | Step-by-step lessons, guided learning, builds confidence | "Getting Started", "Your First Project", "Quickstart" |
| **HOW-TO** | Solve a problem | Practical + Task | Goal-oriented steps, problem-solving, real scenarios | "Deploy on AWS", "Configure Auth", "Troubleshoot X" |
| **EXPLANATION** | Understand concepts | Information + Understanding | Why things work, context, background, architecture | "How X Works", "Architecture Overview", "Key Concepts" |
| **REFERENCE** | Look up facts | Information + Lookup | Technical descriptions, complete specs, precise details | "API Reference", "CLI Commands", "Configuration Options" |

### Decision Tree

Ask: **"What is the user trying to DO?"**

1. **Learn by doing a guided lesson** → TUTORIAL
   - They're new and need to build confidence
   - They need a complete working example
   - Focus: Learning-oriented, step-by-step

2. **Accomplish a specific task** → HOW-TO
   - They have a goal and need steps to achieve it
   - They want to solve a specific problem
   - Focus: Task-oriented, practical

3. **Understand how/why something works** → EXPLANATION
   - They need context and conceptual understanding
   - They want to know the "why" behind decisions
   - Focus: Understanding-oriented, conceptual

4. **Look up specific information** → REFERENCE
   - They know what they're looking for
   - They need complete, precise specifications
   - Focus: Information-oriented, comprehensive

## Quality Principles

### 1. High Signal-to-Noise Ratio

**Every sentence must earn its place.**

✅ **Do**:
- Start with most important information (inverted pyramid)
- Use concrete examples with real code/commands
- Provide specific outcomes and metrics
- Reference source code when relevant
- Remove unnecessary words

❌ **Don't**:
- Use marketing language ("powerful", "flexible", "easy", "robust")
- Use vague terms ("many", "various", "different approaches")
- Write abstract descriptions without examples
- Use paragraphs when lists suffice

**Test**: "If I removed this sentence, would the user lose important information?"
- No → Remove it
- Yes → Keep it

### 2. Progressive Disclosure

Layer information so users get what they need when they need it:

- **Layer 1 (30 seconds)**: What is it, who it's for, key value (first 2-3 paragraphs)
- **Layer 2 (3-5 minutes)**: Core concepts, common use cases, basic patterns (main body)
- **Layer 3 (10+ minutes)**: Advanced features, edge cases (dropdowns/tabs/separate pages)
- **Layer 4 (as needed)**: Complete specs, all options (always separate pages)

### 3. Single Purpose Per Page

Each page should fit ONE Diataxis quadrant. Don't mix tutorials with reference tables, or how-tos with architecture explanations. Use cross-links to related content types.

## Content Type Templates

### Tutorial Structure

```markdown
# [Tutorial Title]

**What you'll learn**: [Objectives]
**Time**: [Estimate]
**Prerequisites**: [Required]

## Step 1: [Action]
[Guided instruction with explanation]

## Step 2: [Action]
[Continue...]

## Next Steps
[Where to go after]
```

**Key characteristics**:
- Learning-oriented, builds confidence
- Step-by-step with explanations
- Complete working examples
- Expected outcomes at each step

### How-To Structure

```markdown
# How to [Goal]

**Goal**: [What you'll accomplish]
**Prerequisites**: [Required]

## Steps
1. [Action] - [Expected outcome]
2. [Action] - [Expected outcome]

## Troubleshooting
[Common issues]

## Related
[Other how-tos]
```

**Key characteristics**:
- Task-oriented, solves specific problem
- Practical steps to achieve goal
- Includes troubleshooting
- Real-world scenarios

### Explanation Structure

```markdown
# [Concept/System]

**Purpose**: [Why this exists]

For hands-on: [Tutorial link]
For tasks: [How-to link]

## How It Works
[Conceptual explanation]

## Key Components
[Detailed breakdown]

## Related
[Cross-references]
```

**Key characteristics**:
- Understanding-oriented, conceptual
- Explains "why" and "how it works"
- Context and background
- No step-by-step instructions

### Reference Structure

```markdown
# [Reference Title]

**Purpose**: [What info is here]

For context: [Explanation link]

## [Category 1]
[Comprehensive, scannable listings]
```

**Key characteristics**:
- Information-oriented, comprehensive
- Scannable format (tables, lists)
- Precise and complete
- Technical specifications

## UX Patterns

### Tab Sets

Use tab sets for **parallel alternatives or variants**:

**Pattern 1: Platform/Language Variants**

```markdown
:::::{tab-set}

::::{tab-item} Python SDK
[Python example]
::::

::::{tab-item} CLI
[CLI example]
::::

:::::
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NVIDIA-NeMo/nemo-platform](https://github.com/NVIDIA-NeMo/nemo-platform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
