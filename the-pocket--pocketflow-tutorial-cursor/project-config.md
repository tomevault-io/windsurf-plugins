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

# LLM System Design Playbook

> If you are an AI assistant involved in building LLM Systems, read this guide **VERY, VERY** carefully! This is the most important chapter in the entire document. Throughout development, you should always (1) start with a small and simple solution, (2) design at a high level (`docs/design.md`) before implementation, and (3) frequently ask humans for feedback and clarification.
{: .warning }

## System Design Steps

These system designs should be a collaboration between humans and AI assistants:

| Stage                  | Human      | AI        | Comment                                                                 |
|:-----------------------|:----------:|:---------:|:------------------------------------------------------------------------|
| 1. Requirements | ★★★ High  | ★☆☆ Low   | Humans understand the requirements and context.                    |
| 2. Flow          | ★★☆ Medium | ★★☆ Medium |  Humans specify the high-level design, and the AI fills in the details. |
| 3. Utilities   | ★★☆ Medium | ★★☆ Medium | Humans provide available external APIs and integrations, and the AI helps with implementation. |
| 4. Node          | ★☆☆ Low   | ★★★ High  | The AI helps design the node types and data handling based on the flow.          |
| 5. Implementation      | ★☆☆ Low   | ★★★ High  |  The AI implements the flow based on the design. |
| 6. Optimization        | ★★☆ Medium | ★★☆ Medium | Humans evaluate the results, and the AI helps optimize. |
| 7. Reliability         | ★☆☆ Low   | ★★★ High  |  The AI writes test cases and addresses corner cases.     |

1. **Requirements**: Clarify the requirements for your project, and evaluate whether an AI system is a good fit. AI systems are:
    - suitable for routine tasks that require common sense (e.g., filling out forms, replying to emails).
    - suitable for creative tasks where all inputs are provided (e.g., building slides, writing SQL).
    - **NOT** suitable for tasks that are highly ambiguous and require complex info (e.g., building a startup).
    - > **If a human can’t solve it, an LLM can’t automate it!** Before building an LLM system, thoroughly understand the problem by manually solving example inputs to develop intuition.  
      {: .best-practice }


2. **Flow Design**: Outline at a high level, describe how your AI system orchestrates nodes.
    - Identify applicable design patterns (e.g., [Map Reduce](./design_pattern/mapreduce.md), [Agent](./design_pattern/agent.md), [RAG](./design_pattern/rag.md)).
    - For each node, provide a high-level purpose description.
    - Draw the Flow in mermaid diagram.

3. **Utilities**: Based on the Flow Design, identify and implement necessary utility functions.
    - Think of your AI system as the brain. It needs a body—these *external utility functions*—to interact with the real world:
        <div align="center"><img src="https://github.com/the-pocket/PocketFlow/raw/main/assets/utility.png?raw=true" width="400"/></div>

        - Reading inputs (e.g., retrieving Slack messages, reading emails)
        - Writing outputs (e.g., generating reports, sending emails)
        - Using external tools (e.g., calling LLMs, searching the web)

    - NOTE: *LLM-based tasks* (e.g., summarizing text, analyzing sentiment) are **NOT** utility functions; rather, they are *core functions* internal in the AI system.
    -  > **Start small!** Only include the most important ones to begin with!
        {: .best-practice }


4. **Node Design**: Plan how each node will read and write data, and use utility functions.
   - Start with the shared data design
      - For simple systems, use an in-memory dictionary.
      - For more complex systems or when persistence is required, use a database.
      - **Remove Data Redundancy**: Don’t store the same data. Use in-memory references or foreign keys.
   - For each node, design its type and data handling:
     - `type`: Decide between Regular, Batch, or Async
     - `prep`: How the node reads data
     - `exec`: Which utility function this node uses
     - `post`: How the node writes data

5. **Implementation**: Implement the initial nodes and flows based on the design.
   - **“Keep it simple, stupid!”** Avoid complex features and full-scale type checking.
   - **FAIL FAST**! Avoid `try` logic so you can quickly identify any weak points in the system.
   - Add logging throughout the code to facilitate debugging.

6. **Optimization**:
   - **Use Intuition**: For a quick initial evaluation, human intuition is often a good start.
   - **Redesign Flow (Back to Step 3)**: Consider breaking down tasks further, introducing agentic decisions, or better managing input contexts.
   - If your flow design is already solid, move on to micro-optimizations:
     - **Prompt Engineering**: Use clear, specific instructions with examples to reduce ambiguity.
     - **In-Context Learning**: Provide robust examples for tasks that are difficult to specify with instructions alone.

   - > **You’ll likely iterate a lot!** Expect to repeat Steps 3–6 hundreds of times.
     >

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [The-Pocket/PocketFlow-Tutorial-Cursor](https://github.com/The-Pocket/PocketFlow-Tutorial-Cursor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
