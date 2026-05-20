---
trigger: always_on
description: @project-documentation(projectName: "open-deep-research - Ultimate Development Guide & Code Snippets Collection") {
---

# .cursorrules

@project-documentation(projectName: "open-deep-research - Ultimate Development Guide & Code Snippets Collection") {

  @section(name: "Project Overview", level: 1) {
    @project-overview {
      @short-description: "**open-deep-research: Your AI-Powered Research Assistant.**  Conduct iterative, deep research using search engines, web scraping, and Gemini LLMs, all within a lightweight and understandable codebase."
      @mcp-tool-availability: "**Seamlessly Integrate with AI Agents via MCP.**  Available as a Model Context Protocol (MCP) tool for easy integration into larger AI agent systems."
      @core-libraries: "**Powered by Key Libraries.** Leverages Firecrawl for efficient web data extraction and Gemini for advanced language understanding and report generation."
      @goal: "**Keep it Simple, Keep it Deep.**  Provides the *simplest* yet *most effective* implementation of a deep research agent, designed for clarity and easy extension (<500 LoC goal). "
      @workflow-reference: "**Workflow Diagram Included.**  Refer to the 'Project Workflow Diagram' section for a visual representation of the research process."
      @license: "**MIT Licensed.**  Freely use, modify, and build upon open-deep-research under the permissive MIT License."
    }
    @note: "**Key Project Philosophy:** 'open-deep-research' prioritizes simplicity and clarity, aiming to provide a foundational research agent that is easy to understand, modify, and extend.  It's designed to be a starting point for building more sophisticated AI-driven research tools."
  }

  @section(name: "Project Workflow Diagram", level: 1) {
    @workflow-diagram(description: "Mermaid flowchart representation of the Deep Research workflow (see README)") {
      @flowchart-mermaid {
        ```mermaid
        flowchart TB
            subgraph Input
                Q[User Query]
                B[Breadth Parameter]
                D[Depth Parameter]
            end

            DR[Deep Research] -->
            SQ[SERP Queries] -->
            PR[Process Results]

            subgraph Results[Results]
                direction TB
                NL((Learnings))
                ND((Directions))
            end

            PR --> NL
            PR --> ND

            DP{depth > 0?}

            RD["Next Direction:
            - Prior Goals
            - New Questions
            - Learnings"]

            MR[Markdown Report]

            %% Main Flow
            Q & B & D --> DR

            %% Results to Decision
            NL & ND --> DP

            %% Circular Flow
            DP -->|Yes| RD
            RD -->|New Context| DR

            %% Final Output
            DP -->|No| MR

            %% Styling
            classDef input fill:#7bed9f,stroke:#2ed573,color:black
            classDef process fill:#70a1ff,stroke:#1e90ff,color:black
            classDef recursive fill:#ffa502,stroke:#ff7f50,color:black
            classDef output fill:#ff4757,stroke:#ff6b81,color:black
            classDef results fill:#a8e6cf,stroke:#3b7a57,color:black

            class Q,B,D input
            class DR,SQ,PR process
            class DP,RD recursive
            class MR output
            class NL,ND results
        ```
      }
      @note: "**Workflow Visualization:** This Mermaid diagram provides a visual overview of the core research process within 'open-deep-research'.  Use it to understand the flow of data and control within the agent."
    }
  }

  @section(name: "Key Features", level: 1) {
    @features-section(description: "Key features of the open-deep-research agent") {
      @feature(name: "MCP Integration", description: "**MCP Ready:** Seamlessly integrates as a Model Context Protocol tool into AI agent ecosystems, enabling plug-and-play research capabilities.")
      @feature(name: "Iterative Research", description: "**Iterative Deep Dive:**  Explores topics deeply through iterative query refinement and result processing, mimicking the in-depth approach of expert human researchers.")
      @feature(name: "Intelligent Query Generation", description: "**Gemini-Powered Queries:**  Leverages the power of Gemini LLMs to generate smart, targeted search queries, adapting to research goals and accumulated learnings for optimal information retrieval.")
      @feature(name: "Depth & Breadth Control", description: "**Tuneable Research Scope:** Offers highly configurable depth and breadth parameters, allowing users to precisely control the scope and intensity of research exploration, from focused investigations to broad surveys.")
      @feature(name: "Smart Follow-up Questions", description: "**Clarify Research Needs with Follow-up Questions:**  Intelligently generates follow-up questions to refine ambiguous user queries, ensuring the research agent is precisely aligned with the user's intended topic.")
      @feature(name: "Comprehensive Markdown Reports", description: "**Detailed, Ready-to-Use Markdown Reports:**  Generates well-structured, human-readable Markdown reports, summarizing key findings, insights, and providing a clear list of sources for verification and further exploration.")

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ssdeanx/deep-research-mcp-server](https://github.com/ssdeanx/deep-research-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
