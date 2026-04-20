---
trigger: always_on
description: Shared writing style rules: tone, rhythm, AI tell avoidance, math vs narrative modes
---


# Writing Style Rules

Shared writing style rules for all textbook chapter workflows (research, write, edit, update).

**Target reader:** Someone with strong reading comprehension and technical background who is new to this specific topic. Write with depth and precision, not oversimplification.

---

## Engaging Writing (Make It Unputdownable)

The best technical textbooks (MacKay, Sutton & Barto, Feynman) are engaging because they use techniques from great non-fiction writing. Apply these:

### Conversational Tone (Like Explaining to a Smart Friend)

- Write as if you're having a conversation with the reader
- Address the reader as "you" directly
- Show enthusiasm: "This is the beautiful part." "Here's where it gets interesting."
- Acknowledge when something is confusing: "This trips up everyone at first."
- Be intellectually honest about uncertainty: "We don't fully understand why, but..."

### Sentence Rhythm and Variety (Gary Provost's Principle)

**Vary sentence length deliberately.** Short sentences punch. Long sentences build momentum and carry the reader through complex ideas with the energy of a crescendo.

- **Short sentences for emphasis:** "That's wrong." "Here's why." "This matters."
- **Medium sentences for flow:** Carry the main explanation forward.
- **Long sentences for buildup:** Where appropriate, build momentum.
- **Deliberate fragments for punch:** "Music." "Exactly." "Finally."

### Micro-Surprises and Dopamine Hits

Keep readers engaged with small rewards throughout:

| Technique | Example |
|---|---|
| **Surprising facts** | "You might expect X, but actually Y" |
| **Rhetorical questions** | "But wait — how can that be?" |
| **Vivid analogies** | "Like untangling earbuds — tedious, but oddly revealing" |
| **Pattern interrupts** | Start a section with something unexpected |
| **Open loops** | Tease what's coming: "We'll see why this matters in Section 3" |
| **Enthusiasm markers** | "This is where it gets good." |

### Motivation Before Formalism

Always answer "why should I care?" BEFORE "how does it work?"

**WRONG (formalism first):**
> Definition: A confidence interval is a range of values, derived from sample statistics...

**RIGHT (motivation first):**
> Imagine you measure the heights of 100 people and get an average of 170 cm. But you know that's not *exactly* the true average — you just happened to measure these 100 people. A confidence interval gives you a range...

---

## Basic Style Rules

**One idea per paragraph:**
- Each paragraph should introduce ONE idea and explain it fully
- If you catch yourself writing "additionally" or "moreover" mid-paragraph, you probably need a new paragraph
- If a paragraph has multiple ideas, split it and add a bridging sentence between the new paragraphs
- Target 2-5 sentences per paragraph; 6+ is a warning sign

**Sentence clarity:**
- Maximum two clauses joined by a comma. Never chain three or more clauses ("X, which Y, allowing Z")
- Keep subject and verb close — do not front-load long modifiers before the main verb
- Make the grammatical subject of each sentence the *agent* doing the action, and make the verb *be* the action itself. This is more general than "prefer active voice." A sentence like "The optimization performs gradient updates" is active voice but still unclear because "the optimization" is not really doing anything; the real character is the algorithm or the learner. Write "The algorithm updates the weights via gradient descent."
- Prefer 15-25 words per sentence. Long sentences (25-35 words) only when building momentum, maximum one per paragraph

**Given-New Contract (information flow):**

Begin each sentence with information the reader already knows (the *topic position*), and end with new, important information (the *stress position*). The new info in sentence N becomes the familiar info in sentence N+1. This creates a chain that pulls the reader forward. (See Gopen & Swan, "The Science of Scientific Writing," *American Scientist* 1990; Williams & Bizup, *Style: Lessons in Clarity and Grace*.)

- **BAD (new info first, context buried at end):** "A 12-layer Transformer with learned positional embeddings processes the resulting sequence of patch tokens. The Vision Transformer introduced this architecture."
- **GOOD (old info first, new info at end):** "The Vision Transformer processes images as sequences of patches. Each patch is projected into a token, and the resulting sequence is fed to a 12-layer Transformer with learned positional embeddings."

**Pronoun clarity ("this + noun" rule):**

Never use bare "this," "that," "it," or "these" as a sentence subject when the antecedent is ambiguous. Always follow the demonstrative with a clarifying noun (a *shell noun*): "this constraint," "this approach," "this result," "these gradients." Research shows "this + shell noun" accounts for over 40% of demonstrative usage in academic writing because it resolves ambiguity that bare "this" creates.

- **BAD:** "We compute the gradient and clip it to a maximum norm. This is then used to update the weights." (What is "this"? The gradient? The clipping? The computation?)
- **GOOD:** "We compute the gradient and clip it to a maximum norm. This clipped gradient is then used to update the weights."


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AI-Learning-Gems) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
