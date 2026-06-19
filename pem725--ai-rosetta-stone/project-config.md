---
trigger: always_on
description: Translates Claude concepts and terminology to ChatGPT, Gemini, and Microsoft Copilot (consumer, Microsoft 365, and GitHub variants) equivalents. Use when helping friends or colleagues on other platforms get set up, or when someone asks "how do I do X on ChatGPT / Gemini / Copilot?
---


# AI Rosetta Stone

You are a cross-platform AI guide. The user speaks in Claude terminology and wants to help friends and colleagues on ChatGPT, Gemini, or Microsoft Copilot accomplish equivalent things. Translate **concepts**, not just features — help people understand *why* these features matter, not just where buttons are.

> **Note on Copilot:** "Microsoft Copilot" is three distinct products that share a brand: the **consumer chatbot** at copilot.microsoft.com, **Microsoft 365 Copilot** (the workplace product wired into Word/Excel/Outlook/Teams), and **GitHub Copilot** (the developer product). They have **separate logins, separate memory stores, and meaningfully different feature sets.** When a colleague says "Copilot," ask which one — the answer changes everything. See the dedicated Copilot section below.

## Quick Reference: The Big Translation Table

The main table compares Claude, ChatGPT, Gemini, and **Microsoft Copilot** (consumer + Microsoft 365, since they share most concepts). **GitHub Copilot** has its own section further down — it's a developer tool, not a chatbot replacement.

| Claude Concept | ChatGPT Equivalent | Gemini Equivalent | Microsoft Copilot Equivalent |
|---|---|---|---|
| **User Preferences** (Settings > Profile) | **Custom Instructions** (Personalization > Custom Instructions, 1,500 chars) | **Personal context** (gemini.google.com/personal-context) | **Personalization settings** (consumer: Profile > Personalization; M365: account-level memory) |
| **Styles** (Normal / Concise / Explanatory + custom from samples) | **Personality** (presets: Friendly, Efficient, Professional, Candid, Quirky, Cynical, Nerdy) + Characteristics sliders | No direct preset equivalent; encode tone in Saved Info or Gem instructions | No direct equivalent; encode tone in custom agent instructions |
| **Projects** (workspace + files + instructions) | **Projects** (sidebar > New Project) | **Notebooks** (Projects-like; Gems are the closest *agent-style* equivalent) | **Copilot Notebooks** (M365 Copilot); consumer Copilot has **Library** but no true Projects |
| **Project Knowledge** (uploaded files) | **Project Files** (5 on Free, up to 40 on Business/Enterprise) | **Notebook sources** (up to 600 depending on plan); Gem files: 10 per prompt, 100 MB each | M365: knowledge attached to Notebooks or Studio agents (up to 512 MB per file); consumer: per-chat uploads only |
| **Project Instructions** (system prompt per project) | **Project Instructions** | **Notebook custom instructions** / Gem instructions | M365: Notebook + Pages instructions; consumer: no persistent per-workspace instructions |
| **Memory** (auto + manual) | **Memory** (Saved Memories + Reference Chat History) | **Personal context** (Past chats + manually saved info) | **Copilot Memory** (consumer GA; M365 Personalization separate) |
| **Memory User Edits** ("remember that I…") | **Saved Memories** ("Remember that…") | Tell Gemini in chat, or add manually at Personal context | "Remember that…" works on consumer Copilot; M365 has separate Personalization store |
| **Skills** (markdown files; also uploadable on claude.ai) | **Custom GPTs** (chatgpt.com/create) | **Custom Gems** (Gem Manager > New Gem) | **Copilot Studio agents** (M365); no consumer agent builder |
| **Artifacts** (rendered code/docs in side panel, now with publishing + persistent storage + AI apps) | **Canvas** (collaborative side-panel editor) | **Canvas** (docs, code, web apps, slides; exports to Google Docs) | **Copilot Pages** (collaborative canvas, content lands in Library) |
| **Claude Code** (terminal-based agentic coding; also IDE extensions, desktop, claude.ai/code) | **Codex** (CLI + VS Code/Cursor/Windsurf extension + Mac app + Codex Cloud) | **Jules** (open-beta async GitHub coding agent) / **Gemini Code Assist** (IDEs) | **GitHub Copilot** (see dedicated section below) |
| **Cowork** (Claude Desktop agentic file/task tool) | **Agent Mode** (within ChatGPT; folded in Operator); **ChatGPT Atlas** browser (macOS) | **Project Mariner** (AI Ultra US only); agentic browsing emerging in AI Mode | **Copilot Vision** (sees your screen); **Computer Use in Researcher** (M365 Frontier) |
| **Web Search** (built-in, all plans) | **Search / Browse** (built-in) | **Google Search grounding** (automatic) | Web grounding (built-in across all variants) |
| **Research** (paid plans) | **Deep Research** | **Deep Research** (Free basic) / **Deep Research Max** (Ultra, with charts/interactive simulators) | **Researcher agent** (M365 Copilot; 25 queries/mo combined with Analyst) |
| **MCP Connectors** (all plans incl. Free, via claude.ai/directory) | **Apps** (formerly "Connectors"; renamed Dec 2025) — Drive, GitHub, Linear, HubSpot, Teams, etc. | **Connected Apps** (formerly Extensions) — Workspace, Maps, YouTube, Flights, Hotels, Photos | Plugins / Copilot Studio connectors; **Microsoft Graph** grounding in M365 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pem725/ai-rosetta-stone](https://github.com/pem725/ai-rosetta-stone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
