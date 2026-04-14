---
trigger: always_on
description: ================================================
---

================================================
File: docs/guide.md
================================================
---
layout: default
title: "Build your LLM App"
---

# LLM Application Development Playbook

> If you are an AI assistant involved in building LLM Apps, read this guide **VERY, VERY** carefully! This is the most important chapter in the entire document. Throughout development, you should always (1) start with a small and simple solution, (2) design at a high level (`docs/design.md`) before implementation, and (3) frequently ask humans for feedback and clarification.
{: .warning }

## System Design Steps

These system designs should be a collaboration between humans and AI assistants:

| Stage                  | Human      | AI        | Comment                                                                 |
|:-----------------------|:----------:|:---------:|:------------------------------------------------------------------------|
| 1. Project Requirements | ★★★ High  | ★☆☆ Low   | Humans understand the requirements and context best.                    |
| 2. Utility Functions   | ★★☆ Medium | ★★☆ Medium | The human is familiar with external APIs and integrations, and the AI assists with implementation. |
| 3. Flow Design         | ★★☆ Medium | ★★☆ Medium | The human identifies complex and ambiguous parts, and the AI helps with redesign. |
| 4. Data Schema         | ★☆☆ Low   | ★★★ High  | The AI assists in designing the data schema based on the flow.          |
| 5. Implementation      | ★☆☆ Low   | ★★★ High  | The human identifies complex and ambiguous parts, and the AI helps with redesign. |
| 6. Optimization        | ★★☆ Medium | ★★☆ Medium | The human reviews the code and evaluates the results, while the AI helps optimize. |
| 7. Reliability         | ★☆☆ Low   | ★★★ High  | The AI helps write test cases and address corner cases.                 |

1. **Project Requirements**: Clarify the requirements for your project, and evaluate whether an AI system is a good fit. An AI systems are:
    - suitable for routine tasks that require common sense (e.g., filling out forms, replying to emails).
    - suitable for creative tasks where all inputs are provided (e.g., building slides, writing SQL).
    - **NOT** suitable for tasks that are highly ambiguous and require complex information (e.g., building a startup).
    - > **If a human can’t solve it, an LLM can’t automate it!** Before building an LLM system, thoroughly understand the problem by manually solving example inputs to develop intuition.  
      {: .best-practice }

2. **Utility Functions**: AI system is the decision-maker and relies on *external utility functions* to:

   <div align="center"><img src="https://github.com/the-pocket/PocketFlow/raw/main/assets/utility.png?raw=true" width="400"/></div>

   - Read inputs (e.g., retrieving Slack messages, reading emails)
   - Write outputs (e.g., generating reports, sending emails)
   - Use external tools (e.g., calling LLMs, searching the web)
   - In contrast, *LLM-based tasks* (e.g., summarizing text, analyzing sentiment) are **NOT** utility functions. Instead, they are *internal core functions* within the AI system—designed in step 3—and are built on top of the utility functions.
   -  > **Start small!** Only include the most important ones to begin with!
      {: .best-practice }

3. **Flow Design (Compute)**: Create a high-level outline for your application’s flow.
    - Identify potential design patterns (e.g., Batch, Agent, RAG).
    - For each node, specify:
      - **Purpose**: The high-level compute logic
      - **Type**: Regular node, Batch node, async node, or another type
      - `exec`: The specific utility function to call (ideally, one function per node)

4. **Data Schema (Data)**: Plan how data will be stored and updated.
   - For simple apps, use an in-memory dictionary.
   - For more complex apps or when persistence is required, use a database.
   - For each node, specify:
     - `prep`: How the node reads data
     - `post`: How the node writes data

5. **Implementation**: Implement nodes and flows based on the design.
   - Start with a simple, direct approach (avoid over-engineering and full-scale type checking or testing). Let it fail fast to identify weaknesses.
   - Add logging throughout the code to facilitate debugging.

6. **Optimization**:
   - **Use Intuition**: For a quick initial evaluation, human intuition is often a good start.
   - **Redesign Flow (Back to Step 3)**: Consider breaking down tasks further, introducing agentic decisions, or better managing input contexts.
   - If your flow design is already solid, move on to micro-optimizations:
     - **Prompt Engineering**: Use clear, specific instructions with examples to reduce ambiguity.
     - **In-Context Learning**: Provide robust examples for tasks that are difficult to specify with instructions alone.

   - > **You’ll likely iterate a lot!** Expect to repeat Steps 3–6 hundreds of times.
     >
     > <div align="center"><img src="https://github.com/the-pocket/PocketFlow/raw/main/assets/success.png?raw=true" width="400"/></div>
     {: .best-practice }

7. **Reliability**  

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/The-Pocket) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
