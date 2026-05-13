---
trigger: always_on
description: This document outlines the implementation plan for conducting deep research using the methodologies from deepresearchprocess.md and the Graph of Thoughts (GoT) framework.
---

# Deep Research Implementation Plan

## Overview
This document outlines the implementation plan for conducting deep research using the methodologies from deepresearchprocess.md and the Graph of Thoughts (GoT) framework.

## Understanding Deep Research

Deep research is an AI-driven approach that autonomously conducts multi-step research by:
- Iteratively searching, reading, and analyzing information
- Performing multi-step reasoning
- Compiling findings with explicit citations
- Using Graph of Thoughts for complex problem-solving

## The 7-Phase Deep Research Process

### Phase 1: Question Scoping
- Clarify the research question with the user
- Define output format and success criteria
- Identify constraints and desired tone
- Create unambiguous query with clear parameters

### Phase 2: Retrieval Planning
- Break main question into subtopics
- Generate specific search queries
- Select appropriate data sources
- Create research plan for user approval
- Use GoT to model the research as a graph of operations

### Phase 3: Iterative Querying
- Execute searches systematically
- Navigate and extract relevant information
- Formulate new queries based on findings
- Use multiple search modalities (web search, file analysis, etc.)
- Apply GoT operations for complex reasoning

### Phase 4: Source Triangulation
- Compare findings across multiple sources
- Validate claims with cross-references
- Handle inconsistencies
- Assess source credibility
- Use GoT scoring functions to evaluate information quality

### Phase 5: Knowledge Synthesis
- Structure content logically
- Write comprehensive sections
- Include inline citations for every claim
- Add data visualizations when relevant
- Use GoT to optimize information organization

### Phase 6: Quality Assurance
- Check for hallucinations and errors
- Verify all citations match content
- Ensure completeness and clarity
- Apply Chain-of-Verification techniques
- Use GoT ground truth operations for validation

### Phase 7: Output & Packaging
- Format for optimal readability
- Include executive summary
- Create proper bibliography
- Export in requested format

## Graph of Thoughts Integration

The GoT framework enhances deep research by:
1. **Modeling Research as Graph Operations**: Each research step becomes a node in the graph
2. **Parallel Processing**: Multiple research paths can be explored simultaneously
3. **Scoring & Optimization**: Information quality can be scored and optimized
4. **Backtracking**: Poor research paths can be abandoned for better alternatives

### GoT Operations for Deep Research:
- **Generate**: Create search queries and hypotheses
- **Score**: Evaluate information quality and relevance
- **GroundTruth**: Verify facts against authoritative sources
- **Aggregate**: Combine findings from multiple sources
- **Improve**: Refine research questions based on findings

## Implementation Tools

### Core Tools:
1. **WebSearch**: Built-in web search capability for finding relevant sources
2. **WebFetch**: For extracting and analyzing content from specific URLs
3. **Read/Write**: For managing research documents locally
4. **Task**: For spawning autonomous agents for complex multi-step operations
5. **TodoWrite/TodoRead**: For tracking research progress

### MCP Server Tools:
1. **mcp__filesystem__**: File system operations (read, write, search files)
2. **mcp__puppeteer__**: Browser automation for dynamic web content
   - Navigate to pages requiring JavaScript
   - Take screenshots of web content
   - Extract data from interactive websites
   - Fill forms and interact with web elements

### Web Research Strategy:
- **Primary**: Use WebSearch tool for general web searches
- **Secondary**: Use WebFetch for extracting content from specific URLs
- **Advanced**: Use mcp__puppeteer__ for sites requiring interaction or JavaScript rendering
- **Note**: When MCP web fetch tools become available, prefer them over WebFetch as per documentation

### Data Analysis:
- Python code execution for data processing
- Visualization tools for creating charts/graphs
- Statistical analysis for quantitative research

## Multi-Agent Research Strategy

### Overview
To maximize research efficiency and coverage, spawn multiple Task agents to work on different aspects of the research simultaneously. This parallel approach mirrors how a research team would divide work among specialists.

### Agent Deployment Strategy

#### Phase 2 (Retrieval Planning) - Agent Distribution:
1. **Topic Decomposition Agent**: Break main question into 3-5 subtopics
2. **Launch Parallel Research Agents**: 
   - One agent per subtopic/research angle
   - Each agent gets specific research objectives
   - Agents work independently but share findings

#### Phase 3 (Iterative Querying) - Parallel Execution:

**Agent Type 1: Web Research Agents** (3-5 agents)
- Focus: Current information, trends, news
- Objective: Gather recent developments and real-world data
- Output: Structured summaries with source URLs

**Agent Type 2: Academic/Technical Agent** (1-2 agents)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AnkitClassicVision/Claude-Code-Deep-Research](https://github.com/AnkitClassicVision/Claude-Code-Deep-Research) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
