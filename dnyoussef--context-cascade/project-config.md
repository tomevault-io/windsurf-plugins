---
trigger: always_on
description: <!-- VCL v3.1.1 COMPLIANT - L1 Internal Documentation -->
---

# CONTEXT CASCADE v3.1.1 :: VERILINGUA x VERIX EDITION

<!-- VCL v3.1.1 COMPLIANT - L1 Internal Documentation -->

## Kanitsal Cerceve (Evidential Frame Activation)
Kaynak dogrulama modu etkin.

---

## L2 DEFAULT OUTPUT RULE (CRITICAL)

[direct|emphatic] ALL user-facing output MUST be L2 compression [ground:vcl-v3.1.1-spec] [conf:0.99] [state:confirmed]

```
L0 (AI<->AI):  A+85:claim_hash     // Maximally compressed
L1 (Internal): [markers] content   // This document format
L2 (User):     Pure English        // ALL responses to user
```

[assert|emphatic] Never output VCL markers to user [ground:system-policy] [conf:0.99] [state:confirmed]

---

## ACTIVE PROJECT: 2025 LIFE AUTOMATION

**Master TODO**: `C:\Users\17175\2025-LIFE-AUTOMATION-TODO.md`

[assert|neutral] 6-stream life automation system [ground:witnessed:planning-session] [conf:0.92] [state:confirmed]

| Stream | Status | Key Skill |
|--------|--------|-----------|
| Content Pipeline | In Progress | cascade-orchestrator + PodBrain |
| Thought Leadership | Pending | visual-art-composition |
| Personal Dashboard | Pending | dashboard-tracking |
| Trader AI | Pending | deployment-readiness |
| Hackathon Automation | Pending | ev-optimizer |
| Fog Compute + Cocoon | Research | literature-synthesis |

---

## IDENTITY DECLARATION

[assert|neutral] Context Cascade cognitive architecture plugin [ground:manifest] [conf:0.99] [state:confirmed]

| Component | Count | Evidence |
|-----------|-------|----------|
| Skills (core) | 176 | [witnessed:file-count] (context-cascade/skills/) |
| Skills (supplementary) | 20 | [witnessed:file-count] (~/.claude/skills/) |
| Skills Total | 196 | [computed:sum] |
| Agents | 260 | [witnessed:file-count] |
| Commands | 249 | [witnessed:file-count] |
| Playbooks | 7 | [witnessed:readme] |

---

## CASCADE DISCOVERY ARCHITECTURE (CRITICAL)

[direct|emphatic] Three-tier discovery system [ground:architecture-design] [conf:0.95] [state:confirmed]

```
User Request
    |
    v
[TOP LEVEL] Skills + Playbooks (SKILL-INDEX.md)
    |         - Route user intent to appropriate skill
    |         - Skills define SOPs (Standard Operating Procedures)
    v
[MID LEVEL] Agents (AGENT-REGISTRY.md)
    |         - Skills invoke agents via Task()
    |         - Agents execute the work
    v
[BASE LEVEL] Commands (COMMAND-INDEX.md)
              - Agents use slash commands
              - Commands bind to skills/actions
```

### Discovery Indexes

| Index | Purpose | Location |
|-------|---------|----------|
| SKILL-INDEX.md | Top-level routing | `discovery/SKILL-INDEX.md` |
| AGENT-REGISTRY.md | Skill->Agent lookup | `discovery/AGENT-REGISTRY.md` |
| COMMAND-INDEX.md | Agent->Command lookup | `discovery/COMMAND-INDEX.md` |
| skill-index.json | Auto-routing data | `scripts/skill-index/skill-index.json` |
| registry.json | Full agent registry | `agents/foundry/registry/registry.json` |

### Slash Commands (Registered)

[assert|neutral] 685 components registered in ~/.claude/commands/ [ground:witnessed:registration] [conf:0.95] [state:confirmed]

```
~/.claude/commands/
  *.md           (249 commands)
  skills/*.md    (176 core skills + 20 supplementary = 196 total)
  agents/*.md    (260 agents)
```

### Auto-Routing Hook

The `skill-router-hook.sh` (UserPromptSubmit) automatically:
1. Parses user intent from message
2. Queries skill-index.json for matches
3. Returns skill suggestions with confidence scores

---

## 5-PHASE WORKFLOW (EVIDENTIAL CHAIN)

[direct|emphatic] Execute on EVERY non-trivial request [ground:system-policy] [conf:0.99] [state:confirmed]

### Phase 1: Intent Analysis
```
Skill("intent-analyzer")
Output: { understood_intent, constraints, confidence, evidence_chain }
```

### Phase 2: Prompt Optimization
```
Skill("prompt-architect")
Output: { optimized_request, added_context, success_criteria }
```

### Phase 3: Strategic Planning
```
Skill("research-driven-planning") OR Skill("planner")
Output: { tasks: [...], execution_order, dependencies }
```

### Phase 4: Playbook Routing
```
Route each task to playbook based on:
- Task type [witnessed:phase-3-output]
- Domain [inferred:keywords]
- Complexity [inferred:scope-analysis]
```

### Phase 5: Execution
```
// Golden Rule: 1 MESSAGE = ALL PARALLEL OPERATIONS
[Single Message]:
  Task("Agent 1", "...", "agent-type-from-registry")
  Task("Agent 2", "...", "agent-type-from-registry")
  TodoWrite({ todos: [...] })
```

---

## AGENT REGISTRY

[assert|neutral] 260 agents across 10 categories [ground:witnessed:file-count] [conf:0.99] [state:confirmed]

```
Category        Count   Evidence
delivery        18      [witnessed:delivery/*.md]
quality         18      [witnessed:quality/*.md]
research        11      [witnessed:research/*.md]
orchestration   21      [witnessed:orchestration/*.md]
security        15      [witnessed:security/*.md]
platforms       12      [witnessed:platforms/*.md]
specialists     45      [witnessed:specialists/*.md]
tooling         24      [witnessed:tooling/*.md]
foundry         18      [witnessed:foundry/*.md]
operations      29      [witnessed:operations/*.md]
```

[direct|emphatic] NEVER use agents not in registry [ground:system-policy] [conf:0.99] [state:confirmed]

---

## SKILL INVOCATION PATTERN


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DNYoussef/context-cascade](https://github.com/DNYoussef/context-cascade) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
