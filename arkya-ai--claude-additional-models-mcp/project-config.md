---
trigger: always_on
description: This project implements a Model Context Protocol (MCP) server that integrates Google Gemini 3.x capabilities directly into the Claude Desktop environment. The primary objective is to create a high-efficiency hybrid intelligence system where Claude acts as the primary orchestrator and Gemini serves as the high-volume execution engine.
---

# CLAUDE.md - Gemini MCP Integration Project Memory

## Project Overview
This project implements a Model Context Protocol (MCP) server that integrates Google Gemini 3.x capabilities directly into the Claude Desktop environment. The primary objective is to create a high-efficiency hybrid intelligence system where Claude acts as the primary orchestrator and Gemini serves as the high-volume execution engine.

### Core Value Proposition
- **10x Reduction in Claude Consumption:** By offloading heavy lifting (long-form writing, large-scale analysis, code generation) to Gemini, Claude's token usage is minimized.
- **90%+ Opus Savings:** High-cost Claude Opus cycles are reserved strictly for complex orchestration and final synthesis, while Gemini handles the bulk of the data processing.
- **Continuous Availability:** Leverages Gemini's high rate limits and cost-effective tiers to ensure the agentic workflow remains responsive even during high-load periods.

### What This Is Not
- **Not a Standalone Chatbot:** This server is designed to be used *by* Claude, not as a replacement for the Claude interface.
- **Not a Bypass for Safety Filters:** All Gemini safety settings are active; this tool does not circumvent provider-level content restrictions.
- **Not a Persistent Database:** While it processes large amounts of data, it does not provide long-term vector storage or RAG memory outside of the current session context.
- **Not a Direct File System Access Tool:** Gemini tools return data to Claude; Claude remains the only entity authorized to write to the local filesystem.

### Quick Start
1. **Prerequisites:** Node.js v18+, Google AI Studio API Key.
2. **Installation:** Clone the repository and run `npm install`.
3. **Build:** Run `npm run build` to generate the distribution files.
4. **Configuration:** Add the server to your `claude_desktop_config.json` (see Configuration section).
5. **Verification:** Open Claude Desktop and look for the Gemini MCP tools. Test with: "Use Gemini to summarize the latest news on MCP."

### Key Metrics
- **Target Token Reduction:** 10:1 ratio (Gemini tokens to Claude tokens).
- **Efficiency Goal:** 90% reduction in Claude-native processing for tasks exceeding 500 words or 100 lines of code.
- **Reliability:** 99% tool call success rate through robust error handling and fallback logic.

---

## Architecture
The integration utilizes a dual-path communication strategy to maximize feature availability while maintaining compatibility.

### Data Flow
1. **User Input:** User provides a prompt to Claude Desktop.
2. **Orchestration:** Claude determines if the task requires high-volume processing or external data.
3. **Tool Call:** Claude sends a JSON-RPC request to the MCP Server.
4. **API Routing:** The MCP Server routes the request to either the OpenAI SDK path or the Native Gemini API path.
5. **Execution:** Gemini processes the request (optionally using Google Search grounding).
6. **Return Path:** Gemini Response -> MCP Server -> Claude Desktop.
7. **Final Synthesis:** Claude reviews the Gemini output and presents it to the user or performs file operations.

### Hybrid Communication Approach
1. **OpenAI SDK Path:** Used for standard chat-based tools (`ask_gemini`, `ask_gemini_pro`). This provides a stable, standardized interface for text-to-text operations and ensures compatibility with standard streaming protocols.
2. **Native Gemini API Path:** Used for advanced features including `google_search` grounding, `url_context`, and multi-modal document processing. This path bypasses the OpenAI compatibility layer to access Gemini-specific parameters like `dynamic_retrieval_config`.

### OpenAI SDK vs. Native API Tradeoffs
- **OpenAI SDK:**
    - *Pros:* Simpler error handling, standardized response format, easier to swap models.
    - *Cons:* No access to Google Search grounding, limited multi-modal support.
- **Native API:**
    - *Pros:* Full access to Gemini 3.x features, grounding, system instructions, and safety settings.
    - *Cons:* More complex request/response structure, requires specific SDK management.

### Model Specifications
- **gemini-3-flash-preview:**
    - **Role:** General purpose, high-speed execution.
    - **Default Output Tokens:** 8,192.
    - **Use Case:** Summarization, basic research, drafting, and sub-agent tasks.
- **gemini-3-pro-preview:**
    - **Role:** Complex reasoning and heavy code generation.
    - **Default Output Tokens:** 16,384.
    - **Use Case:** Architectural design, complex debugging, deep technical analysis.

### Core Toolset
1. `ask_gemini`: General text generation and reasoning via Flash.
2. `ask_gemini_pro`: High-complexity reasoning and code generation via Pro.
3. `web_search`: Real-time information retrieval using Google Search grounding.
4. `web_reader`: High-fidelity content extraction from specific URLs.
5. `parse_document`: Advanced OCR and structured data extraction from PDFs and images.

### Configuration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Arkya-AI/claude-additional-models-mcp](https://github.com/Arkya-AI/claude-additional-models-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-12 -->
