---
trigger: always_on
description: These are methodologies for approaching development work. For the overarching design philosophy, see `philosophy.mdc`. For specific actionable rules, see `core-rules.mdc`.
---


# Development Practices

These are methodologies for approaching development work. For the overarching design philosophy, see `philosophy.mdc`. For specific actionable rules, see `core-rules.mdc`.

---

## Tracer Bullet Development - Building Through the Fog

**Tracer bullet development is about building a thin, end-to-end slice of the system that works from the start.**

Like tracer rounds in combat, it provides real-time feedback on where you are aiming while you are still moving.

**Core idea**: Build something that runs through the entire system early, then evolve it.

**Mental model**: You are aiming in the dark. Tracer bullets show you where your system is actually going.

### The Problem It Solves

In new systems:
- Users don't fully know what they want
- Developers don't fully know what they are building
- Requirements are vague and will change
- Components have never been integrated before

**Depth-first development** (perfecting one layer in isolation):
- Optimizes parts before validating the whole
- Discovers integration problems late
- Creates expensive rework

**Tracer development**:
- Validates architecture early
- Exposes integration issues immediately
- Keeps progress visible

**Mental model**: Don't perfect pieces before you know the whole works.

### What a Tracer Bullet Is

A tracer bullet is:
- **End-to-end**: Passes through all layers
- **Minimal**: Simplest possible implementation
- **Functional**: Actually works, not a mock
- **Built to be extended**: Not thrown away, forms the foundation

**Example**: If your system has Architecture → API → Compute → Database → UI, a tracer bullet might pass a single string or record through all layers, touching every boundary once, proving the system "hangs together."

**Mental model**: Build the skeleton first. Add muscle later.

### Tracer Bullets vs Prototypes

**Prototypes**:
- Built to learn
- Often partial
- Typically disposable
- Explore specific unknowns

**Tracer bullets**:
- Built to keep
- End-to-end
- Architecturally real
- Form the foundation of the final system

**Key distinction**: Prototypes generate knowledge. Tracer bullets generate structure.

**Decision rule**:
- If your goal is "Do these parts hang together?" → use a tracer bullet
- If your goal is "Does this idea even work?" → build a prototype

**Mental model**: Prototype to understand. Tracer to build.

### Why Tracer Bullets Matter

**Benefits**:
- Early user feedback
- Early architectural validation
- Continuous sense of progress
- Reduced risk of late-stage surprises
- Shared concrete reference for the team

**For developers**:
- A real code path to evolve
- A scaffold for adding features
- A living architectural diagram

**For stakeholders**:
- Something demonstrable
- Proof of direction
- Faster alignment

**Mental model**: Seeing something work beats reasoning about what might work.

### Tracer Bullets and Change

Because tracer bullets:
- Use real integrations
- Cross real boundaries
- Expose real constraints

They:
- Reveal mismatches early
- Allow fast course correction
- Prevent over-investment in the wrong direction

**Mental model**: Correct early while the cost of change is still low.

### Tracer Bullets and Other Principles

**DRY**: The tracer establishes the authoritative flow of knowledge

**Decoupling / Orthogonality**: Boundaries become visible and testable

**Reversibility**: Early structure keeps choices changeable before they harden

**Together**: Tracer bullets make architectural decisions real while they are still cheap to change.

**Mental model**: Architecture is not what you draw. It is what your code already does.

### Common Failure Modes

**Not a tracer**:
- "Let's fully design this layer first"
- "We'll integrate later"
- "This is just a mock for now"

**Anti-patterns**:
- Building isolated components with no integration
- Writing throwaway "temporary" code that becomes permanent
- Over-polishing before validating end-to-end flow

**Mental model**: If it doesn't run end-to-end, it doesn't exist yet.

### The Approach

1. Build the simplest possible version that works end-to-end
2. Deploy it to real infrastructure (not mocks)
3. Get feedback, iterate
4. Flesh out features incrementally

### Design Checklist

When starting a system:
1. Can I push a minimal piece of data through the entire stack?
2. Does this exercise touch every major architectural boundary?
3. Is this code written to evolve, not to be discarded?
4. Are users seeing something real early?

**Mental model**: Build a thin working system first. Then grow it deliberately.

**For this project**: Every new feature should start with the thinnest possible working slice.

---

## Prototyping - Learning Before Committing

**Prototyping is about building small, disposable experiments to answer specific questions about a system.**

It is cheaper than full-scale production and exists only to generate insight.

**Core idea**: When something is uncertain, build a quick experiment to learn.

**Mental model**: Prototypes exist to reduce ignorance, not to become the product.

### Why Prototype

In complex systems:
- Some aspects are unproven
- Some behaviors are unknown

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sanchit1591) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
