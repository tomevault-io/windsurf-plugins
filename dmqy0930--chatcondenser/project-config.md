---
trigger: always_on
description: Use this skill when the user wants to convert a DeepSeek web chat (via share link) into a structured review outline for Obsidian knowledge management. It restructures Q&A into a logical knowledge framework, removes conversational noise, and outputs interlinked Markdown notes with wikilinks and YAML frontmatter. Triggered by DeepSeek share links or explicit requests to summarize learning-oriented AI chats.
---


# ChatCondenser — DeepSeek Chat → Obsidian 知识网络

Convert a messy, multi-turn DeepSeek learning conversation into a structured, interlinked Obsidian knowledge network. The output is not a chat log or FAQ list—it is a Map of Content (MOC) main note plus optional concept sub-notes, connected by `[[wikilinks]]` for bidirectional navigation.

## Non-Negotiables

- **REQUIRED COMPANION SKILL:** use `pdf` only for PDF-based source input (e.g., exported chat PDFs, screenshots). For plain share links or pasted text, `pdf` is not needed.
- **NO OTHER SKILLS:** when this skill applies, use only `chatcondenser` plus `pdf`. Do **not** invoke any other skill. Any task-splitting, verification, or dispatch logic is already defined inside this skill.
- **Primary deliverable is `.md` files in the Obsidian vault.** PDF compilation (`xelatex`) is strictly optional—only run it when the user explicitly requests a PDF export.
- When this skill applies, a chat-only or inline summary is **not** a successful final deliverable unless the user explicitly asks for summary-only output or opts out of file creation.
- **All `.md` files must be written to `{vaultPath}/{targetFolder}/`** (as configured in frontmatter metadata). Do not write notes anywhere else.
- The vault path is read from this SKILL.md's `metadata.vaultPath` field. If it is missing or points to a non-existent directory, stop and ask the user to set it.
- The input is primarily a **DeepSeek share link** (e.g., `https://chat.deepseek.com/share/...`). Use `WebFetch` to retrieve its content. If the link cannot be fetched, accept raw pasted text as a fallback.
- The default writing style is **Chinese-first**, with important technical terms followed by their English equivalents in parentheses.
- The output is a **review outline**, not a chronology. The original Q&A format must be completely dissolved and restructured by topic and logic.
- **No star ratings or decorative icons.** Use basic text formatting: bold, italic, `==highlight==`, Obsidian callouts (`> [!note]`), and Markdown tables.
- Every key concept must link to other notes via `[[wikilinks]]` for knowledge networking.
- All notes must include YAML frontmatter with at least `tags` and `created` fields.

## Skill Boundary

- Allowed skills for this workflow: `chatcondenser` and `pdf` only.
- Forbidden: all other skills, even if they appear relevant to planning, dispatch, or verification.

## Required Outcome

The default successful outcome for this skill is:

- a **main note (MOC)** saved as `Chat-<Topic>.md` inside `{vaultPath}/{targetFolder}/`
- zero or more **concept sub-notes** (stubs for independently meaningful concepts), each with bidirectional `[[links]]` back to the MOC
- all notes contain valid YAML frontmatter (`tags`, `created`, `source`) and Obsidian wikilinks
- a final response that reports the written file paths, linking coverage, and any blockers

**Optional:** if the user explicitly requests PDF export, additionally produce `Chat-<Topic>.pdf` via `xelatex` and report the PDF path.

The following are **not** successful completions:
- an inline chat summary only
- a document that still retains a Q&A or conversational structure
- stopping at analysis without writing `.md` files to the vault
- generating files outside `{vaultPath}/{targetFolder}/`
- wikilinks that point to non-existent notes without creating stubs (when `autoCreateStubs: true`)

## When to Use

Use this skill when the user wants to turn a learning chat with DeepSeek into Obsidian revision material. Typical triggers:

- User provides a `chat.deepseek.com/share/...` link and asks to "整理成复习提纲", "提炼知识点", "做成笔记"
- User pastes a long DeepSeek conversation and wants structured, interlinked notes
- User mentions "对话总结", "聊天记录提炼", "把和DeepSeek的对话变成Obsidian笔记"

Do not use this skill for:
- summarizing meetings, interviews, or non-learning conversations
- turning a chat into an FAQ or Q&A list (unless the user explicitly asks)
- processing chats from platforms other than DeepSeek (unless the user explicitly confirms format compatibility)
- producing PDF-only output when the user has not asked for PDF

## Portability

- Do not assume any custom skill other than `pdf` exists.
- This skill reads its own configuration from the `metadata` fields in its YAML frontmatter.
- If subagents are available, prefer them for topic-clustering readers (Step 4) and the verifier (Step 11). If not, execute sequentially.
- `WebFetch` is the preferred method for retrieving DeepSeek share link content.

## Workflow Model

The workflow follows a controller-reader-verifier architecture specialized for conversational content and Obsidian output:

```
Intake → Fetch & Clean → Topic Clustering → Correction Chain →
Wikilink & Split → Restructure & Tables → Glossary → Merge →
Verify → Write .md files → (Optional PDF) → Report
```

## Step-by-Step Execution

---

### Step 1. Intake


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dmqy0930/chatcondenser](https://github.com/dmqy0930/chatcondenser) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
