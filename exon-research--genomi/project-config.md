---
trigger: always_on
description: After installation, use this file for session rules and common operations. Use
---

# Agents Working With Genomi

After installation, use this file for session rules and common operations. Use
`INSTALL_FOR_AGENTS.md` only for setup.

## Connect

Prefer MCP:

```json
{
  "mcpServers": {
    "genomi": {
      "command": "genomi",
      "args": ["serve"]
    }
  }
}
```

Call operations through MCP. Base operations are direct-callable; capability
operations go through `genomi.invoke` after reading the matching skill.

## Operating Rule

Only information discussed in the current chat is current context.
`SKILL.md`, `AGENTS.md`, and focused skill docs are static guidance. Live state
comes from Genomi operations and result envelopes.

- If the user asks a public genetics, variant, gene, phenotype, disease, screen,
  or pharmacogenomics question, answer from public/tool evidence.
- If the user provides a genome source file, use that as the current Active Genome Index
  context. Supplying the file path in this chat is approval to read that source
  for this session.
- If the user names a previous run, use that as the current Active Genome Index
  context only after explicit approval for this session.
- If the user says "my Active Genome Index", "my genome", or similar without a path, you may
  check whether Genomi already has an imported Active Genome Index context to use.
- Reading imported/parsed Active Genome Index artifacts, or searching for an existing
  "my Active Genome Index"/"my genome" context, requires explicit user approval for this
  session. After approval, record it with `active_genome_index.approve_access`.
- Do not use unrelated genome sources from other chats or previous tasks.

## Fresh Task Flow

1. Read `SKILL.md`.
2. Resolve context from this chat. If the user supplied a genome source, prepare or
   select the Active Genome Index before making sample-specific claims.
3. Identify the user's intent and the matching capability. MCP `tools/list`
   returns only the base set (`genomi.*` and `journal.*`) plus the
   `genomi.invoke` dispatcher; every other capability tool is reached by
   reading `skills/<capability>/SKILL.md` first, then calling
   `genomi.invoke` with the registered operation name, for example
   `genomi.invoke({tool: "variant.resolve", params: {"rsid": "rs429358"}})`.
   Do not use the capability ID as the tool name. Anthropic
   Claude Code Skills auto-loads each capability's skill via its YAML
   frontmatter (installed as `~/.claude/skills/genomi-<capability>/`).
4. Base tools (`genomi.*`, `journal.*`) are direct-callable from MCP without
   a skill read.
5. Call the smallest useful Genomi operation for the question.
6. Before calling a tool, only provide parameters supplied by the current user
   request, current Genomi context, a previous Genomi result, explicit user
   approval, or an explicit override. Omit unknown optional parameters.
7. Inspect `defaults_applied` in every result. These defaults are part of the
   reasoning chain; change them in a follow-up call when the user's intent or
   evidence context requires a different assumption.
8. Inspect the returned evidence. When the investigation spans multiple tools or
   produces a material finding, append a journal entry with evidence links.
9. Continue within the same category when the next question is local to that
   evidence. If the evidence shows another category is needed, discover that
   category's tools and repeat.
10. If an MCP tool returns `status="in_progress"`, the operation is still running
   in a background Genomi job. Call `genomi.check_background_job` with the
   returned `job_id`; do not retry with a smaller slice or raw file scan unless
   the user explicitly asks for that fallback.
11. If a tool returns `status="requires_library_install"`, the library is not
   installed and that evidence tool will not work yet. Explain how the named
   library helps the user's intent, ask whether they want it installed, and use
   the returned install command if they approve. Do not treat missing library
   evidence as negative evidence.
12. If a result contains `ask_user`, the presence of that object means the host
   agent should ask. Surface its `question`, and use its `install_command` only
   after approval.
13. Tool definitions expose `dependencyContract` when they need installed
   libraries, local source files, or external network/API sources. External
   source failures return `source_unavailable`; retry later, use another source,
   or state the answerability gap rather than treating it as negative evidence.
14. Mention Active Genome Index use only when it materially affects the result:
   for example, it supports or refutes a user-specific claim, changes a
   limitation, blocks an operation until approval, or explains a required next
   action. Do not add a routine source-status line.
15. Derive answer confidence dynamically from the returned evidence, source
   trust, coverage, conflicts, and missing evidence. Do not use a static
   default confidence or a user-selected confidence profile.
   Genomi result fields describe evidence support, coverage, overlap, and source
   state; they are not final answer-confidence labels.
16. Use `genomi.describe_context` when the user asks about personal context,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [exon-research/genomi](https://github.com/exon-research/genomi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
