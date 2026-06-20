---
trigger: always_on
description: Build validated web research processes through self-annealing loops. Takes a research goal, generates search steps, tests against sample companies, scores accuracy, and iterates until 90%+. Use when creating new research workflows, building claygent/agent prompts, or systematizing any web research task.
---


# Research Process Builder

Build validated, step-by-step web research processes through iterative testing. Takes any research goal, generates search patterns, tests them against real companies, scores accuracy, and loops until the process hits 90%+ reliability.

This is the factory that produces research agent prompts. The output is a portable .md file with step-by-step instructions that any agent (Claude Code, Clay, custom GPT, browser agent) can follow to reliably surface specific intelligence.

## When To Use

- Building a new research workflow for any topic (company intel, market sizing, hiring signals, tech stack detection)
- Creating claygent or web research agent prompts that need to work reliably
- Systematizing any manual web research you do repeatedly
- Someone asks "how do I research X about companies?"

## When NOT To Use

- Running an existing research process (load the process .md directly)
- One-off research where you just need the answer
- Data enrichment at scale (use a dedicated enrichment tool)

---

## Interactive Flow

> Reference: leadgrow-hq/company/methodology/interactive-skill-pattern.md

### Intake

1. **Ask:** "What do you want to research about companies?" (state the research goal in one sentence)
   **Default:** none — REQUIRED
   **Why:** The goal sentence drives pattern generation, scoring criteria, and output template. A vague goal produces vague patterns.

2. **Ask:** "What does a 'good result' look like? What should the output contain?" (3-5 bullet points)
   **Default:** none — REQUIRED
   **Why:** Defines the extraction spec for every search pattern. Without this, Claude can't score Quality — it doesn't know what "good" means for YOUR use case.

3. **Ask:** "Do you have ground truth examples? Companies where you already KNOW the answer, so we can validate accuracy."
   **Default:** no, but strongly encouraged. If yes, collect: company name, domain, and the known-good answer for each (3-5 companies ideal).
   **Why:** Ground truth turns the annealing loop from "does this look right?" to "did we find what we KNOW is there?" Without it, accuracy is subjective. With it, accuracy is measurable.

4. **Ask:** "What accuracy target?"
   **Default:** 90%
   **Why:** Determines when the iteration loop stops. Lower targets finish faster but produce less reliable processes.

5. **Ask:** "Do you have sample companies across size tiers? (enterprise / mid-market / startup)"
   **Default:** suggest 6-10 from existing client list + well-known companies, ensuring Tier 1 (known), Tier 2 (mid), and Tier 3 (obscure) are represented
   **Why:** Patterns that work for SpaceX break for startups. Testing across tiers is what makes the process reliable.

6. **Ask:** "Is this time-sensitive research? (e.g., recent news vs evergreen profiles)"
   **Default:** no (evergreen)
   **Why:** Time-sensitive goals add a Freshness (F) scoring dimension and require `{{current_year}}` variables in all patterns.

7. **Ask:** "Where will this process run? (Claude Code / Clay claygent / browser agent / custom)"
   **Default:** Claude Code
   **Why:** Output format differs — Clay claygents need specific field mappings, browser agents need URL patterns, Claude Code processes are freeform markdown.

### Gap Detection

| Check | Where to Look | If Missing | Severity |
|-------|--------------|------------|----------|
| Research goal is specific enough (not "learn about companies" or "find info") | User input analysis | Ask clarifying questions until goal is one-sentence specific with a clear target | BLOCKING |
| Desired output is concrete (not "useful info") | User's output description | Show examples from existing processes (e.g., find-competitors output spec), ask user to match that specificity | BLOCKING |
| Ground truth variables provided | User input | DEGRADED — can still build, but accuracy validation will be weaker. Suggest: "Can you name 3-5 companies where you already know the answer? This dramatically improves the process." | DEGRADED |
| Sample companies span 3 tiers | User input + existing client list | Auto-suggest from clients and well-known companies. Include at least one ambiguous-name company (Clay, Keep, Harvey). | Auto-resolve |
| Existing process already covers this goal | `research-process-builder/processes/` | Show the existing process, ask: "This already exists. Extend it, or build a new angle?" | BLOCKING |
| Ambiguous-name company included in samples | Sample company list | Add one automatically — ambiguous names stress-test disambiguation logic | Auto-resolve |

### Checkpoints

#### CHECKPOINT 1: Goal + Samples Confirmed

**Show:**
- Formatted research goal (one sentence)
- Desired output spec (bullet points)
- Sample companies organized by tier (Tier 1 / Tier 2 / Tier 3)
- Ground truth variables (if provided) — company name, domain, known answer
- Accuracy target
- Scoring dimensions: Quality + Consistency (+ Freshness if time-sensitive) (+ Accuracy if ground truth provided)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LeadGrowGTM/research-process-builder](https://github.com/LeadGrowGTM/research-process-builder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
