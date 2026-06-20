---
trigger: always_on
description: >
---


# graphifychat

Turn any conversation into a queryable, portable knowledge graph.
Three layers. Every turn captured. Cold-pasteable into any AI.

```
┌─────────────────────────────────────────────────────────────────┐
│  TRON      — one compressed line per turn, always live          │
│  input + output · emotions · psychology · intent · thread       │
├─────────────────────────────────────────────────────────────────┤
│  SPARSE6   — layered relational graph, never loses edges        │
│  turns · concepts · emotions · files · entities · characters    │
│  communities · hyperedges · confidence scores                   │
├─────────────────────────────────────────────────────────────────┤
│  GRAPH_REPORT — god nodes · connections · 4-5 bullet summary    │
│  auto T1-T3 · on demand after                                   │
└─────────────────────────────────────────────────────────────────┘
```

| Layer | Updated | Token cost | Purpose |
|---|---|---|---|
| TRON | Every turn | ~40 tokens/turn | Structured facts: I/O, emotions, psychology, flags |
| Sparse6 | Every turn | ~30 tokens/turn | Relationships TRON can't express as rows |
| GRAPH_REPORT | Auto T1–T3, then on-demand | 0 unless requested | Human + AI readable arc |

---

## Calling Rule

```
/graphifychat [start]     → TRON + Sparse6 update every turn automatically
/graphifychat [mid-conv]  → update only when /graphifychat called again
/graphifychat report      → generate or refresh GRAPH_REPORT now
/graphifychat export      → output full file for copy-paste into another AI
```

If called at the very start of a conversation (T1 or T2), enter **auto mode** — update silently every turn. If called later, enter **manual mode** — update only on explicit call.

---

## File Structure

```
# graphifychat: <topic>
_Turns: <N> | Files: <count> | Open: <count> | Gods: <count> | RES: <1-10> | Updated: <date>_

---

## TRON
\```tron
<one line per turn, T1 → TN>
\```

---

## Sparse6
\```sparse6
<layered adjacency block, 8 layers>
\```

---

## GRAPH_REPORT
<present only if turn ≤ 3 or explicitly requested>
```

Always output the full file as:
````
```md
<file contents>
```
````
Never render. Never outside a code fence. Never partial.

---

## TRON — Full Field Specification

One compressed line per turn. All fields pipe-separated. Omit optional fields only if truly empty.

```
T<N>|U:<keywords>|O:<keywords>|F:<created>|A:<attachments>|IMG:<images>|EMO:<e1>:<i>,<e2>:<i>,<e3>:<i>|BEH:<pattern>|INT:<type>|THR:<id>|CHG:<flag>|OPEN:<yes/no>|SHIFT:<yes/no>|CHAR:<arch>:<i>,<arch>:<i>|CHARSHIFT:<yes/no>|CONF:<tag>|GOD:<concept>
```

### Complete Field Reference

| Field | Required | Auto-trigger | Format | Description |
|---|---|---|---|---|
| `T<N>` | yes | always | `T1`, `T2`… | Turn number |
| `U:` | yes | always | keywords, phrases | **User input** compressed — what they asked, tone, framing |
| `O:` | yes | always | keywords, phrases | **Claude output** compressed — what was produced, result type |
| `F:` | if files created | always | `file.ext` | Files Claude created this turn |
| `A:` | if user attached | always | `doc.pdf` | Docs/files user attached |
| `IMG:` | if images attached | always | `img.png` | Images user attached |
| `EMO:` | yes | always | `emotion:1-5,emotion:1-5,emotion:1-5` | Top 3 user emotions + intensity (from 171-emotion model) |
| `BEH:` | yes | always | single phrase | Dominant behavioral pattern this turn |
| `INT:` | yes | always | see types below | User's intent type this turn |
| `THR:` | yes | always | `a`,`b`,`c`… | Topic thread — groups related turns across the session |
| `CHG:` | yes | always | `+` `~` `!` | Delta type: new / refined / corrected |
| `OPEN:` | yes | always | `yes`/`no` | Was a question or task left unresolved? |
| `SHIFT:` | if EMO delta ≥ 2 | auto | `yes` | Dominant emotion shifted significantly this turn |
| `CHAR:` | yes | always | `archetype:1-5,archetype:1-5` | Top 2 active user character archetypes + intensity |
| `CHARSHIFT:` | if archetype changes | auto | `yes` | Dominant character archetype changed this turn |
| `CONF:` | yes | always | `EXT`/`INF`/`AMB` | Confidence in TRON extraction: Extracted / Inferred / Ambiguous |
| `GOD:` | if god node detected | auto | `concept_name` | A concept introduced this turn that becomes central to the session |

---

### U: and O: — Input/Output Capture

**U: (user input)** — compress the full prompt into keywords and intent phrases. Capture:
- The core ask (what they want done)
- The framing (how they asked — tone, urgency, constraints)
- Any implicit needs (what they didn't say but clearly need)
- Domain keywords, named entities, specific terms they used

**O: (output)** — compress Claude's response into result keywords. Capture:
- What type of output was produced (questions_asked / analysis / code_written / design_confirmed / etc.)
- Key concepts introduced or resolved
- Whether the turn advanced, refined, or corrected prior work

Example:
```
U:compact conv memory,portable,md+tron,track prompts+files+attachments,usable other AIs
O:questions asked,format options presented,two-tier layout proposed
```

---

### EMO: — Emotion Capture (171-emotion model)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [soumyakantabera/graphifychat](https://github.com/soumyakantabera/graphifychat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
