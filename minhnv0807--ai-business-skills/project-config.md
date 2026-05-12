---
trigger: always_on
description: > Huong dan cho bat ky AI agent nao (khong chi Claude) lam viec voi repo nay.
---

# AGENTS.md — Fullstack Marketing Skills

> Huong dan cho bat ky AI agent nao (khong chi Claude) lam viec voi repo nay.

## Tong quan

Repo nay chua **Agent Skills** theo [Agent Skills Specification](https://agentskills.io) — cai dat duoc vao `.agents/skills/` (hoac `.claude/skills/` voi Claude Code).

Repo cung hoat dong nhu **Claude Code Plugin Marketplace** — cai dat bang `/plugin install minhnv0807/ai-business-skills`.

## Cau truc repo

```
.
├── .claude-plugin/
│   └── marketplace.json         # Claude Code plugin manifest
├── .github/
│   ├── ISSUE_TEMPLATE/           # Issue templates
│   └── PULL_REQUEST_TEMPLATE/    # PR templates
├── skills/
│   └── skill-name/
│       ├── SKILL.md              # Skill definition (required)
│       ├── references/           # Supporting docs (optional)
│       ├── scripts/              # Helper scripts (optional)
│       └── evals/                # Quality tests (optional)
├── references/                   # Shared references across skills
├── workflows/                    # Multi-skill workflows
├── agents/                       # Agent personas/roles
├── examples/                     # Sample outputs
├── AGENTS.md                     # This file
├── CLAUDE.md                     # Claude-specific instructions
├── VERSIONS.md                   # Version tracking
├── CONTRIBUTING.md               # Contribution guide
├── validate-skills.sh            # Spec validator (bash)
└── validate-skills.ps1           # Spec validator (PowerShell)
```

## Yeu cau skill

Moi `SKILL.md` phai co:

### 1. YAML Frontmatter

```yaml
---
name: skill-name                    # Required: 1-64 chars, lowercase + hyphens, match directory
description: "Mo ta cu the..."       # Required: 1-1024 chars, co trigger phrases
metadata:
  version: 1.0.0                    # Optional: semver
  category: strategy | content | performance | operations
license: MIT                        # Optional: default MIT
triggers:                           # Optional: keyword list for auto-activation
  - "trigger 1"
  - "trigger 2"
related:                            # Optional: skill cross-references
  - skill-1
  - skill-2
---
```

### 2. Noi dung

- **Duoi 500 dong** — chi tiet chuyen vao `references/`
- Ngon ngu ro rang, active voice
- Cau truc H2 va H3
- Co "Thu thap thong tin" section (toi da 4 cau)
- Co output template co cau truc
- Co quality checklist

## Foundation skill

**`product-marketing-context`** la skill goc — moi skill khac doc truoc khi hoat dong.

File `.agents/product-marketing-context.md` chua:
- Product overview
- Target audience
- Personas
- Pain points
- Competition
- Differentiation
- Objections
- Customer language
- Brand voice
- Proof points
- Goals

Skill khac kiem tra file nay truoc — neu co, lay thong tin san; neu khong, de xuat tao.

## Pattern variants (Skill 20 + Skill 22)

Skills su dung pattern variants:
- `20-brief-client-intake` — 20 variants theo nganh hang
- `22-personal-brand-context` — 3 variants theo nhom audience (founder/coach/creator)

Cau truc:

```
modules/personal-branding/vi/22-personal-brand-context/
├── SKILL.md          ← Entrypoint + router
├── README.md         ← Variant guide (decision tree)
└── variants/
    ├── 01-founder.md
    ├── 02-coach.md
    └── 03-creator.md
```

User chay skill chinh, skill load README → user chon variant → skill load variant template.

### Pattern variants — Skill 22 Global (NEW)

Skill 22-personal-brand-context-global uses 4 REGION variants (US/EU/SEA/LATAM) instead of 3 audience variants like VN.

Cau truc:

```
modules/personal-branding/en/22-personal-brand-context-global/
├── SKILL.md          ← Entrypoint + router
├── README.md         ← Region variant guide
└── variants/
    ├── 01-us.md      ← Covers founder + coach + creator (US)
    ├── 02-eu.md      ← Covers founder + coach + creator (EU)
    ├── 03-sea.md     ← Covers founder + coach + creator (SEA)
    └── 04-latam.md   ← Covers founder + coach + creator (LATAM)
```

Difference from VN skill 22:
- VN: 3 audience variants (founder/coach/creator separately)
- Global: 4 region variants (each contains 3 audience inside)
- Reason: persona universal but currency/platforms/regulations differ per region

## Pattern Mode-Switching (Skills 04, 05)

Skill 04, 05 dung pattern context-aware mode:
- Doc 1 hoac nhieu file `.agents/*.md`
- Auto-detect mode dua tren context file ton tai
- Output adapt theo mode

Pattern dieu kien:
1. Read both context files (skill checks existence)
2. If only A → Mode A
3. If only B → Mode B
4. If both → Skill ASKS user
5. If neither → Skill SUGGESTS creating context first

## Pattern: Cluster Auto-Detect Mode (v2.5.0+)

Pattern used in v2.5.0 to handle 2 clusters (VN + Global) in same agents:

```
Agent reads `.agents/`:
- `product-marketing-context.md` → MODE VN
- `product-marketing-context-global.md` → MODE GLOBAL
- Both → ASK 1 question
- None → SUGGEST creating context
```

Same pattern for personal brand:
- `personal-brand-context.md` → MODE VN PB
- `personal-brand-context-global.md` → MODE GLOBAL PB

This pattern allows ONE agent to serve BOTH clusters without code duplication.

Used by: 5 agents (mkt-strategist, content-producer, performance-analyst, channel-operator, personal-brand-builder).

## Cai dat


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [minhnv0807/ai-business-skills](https://github.com/minhnv0807/ai-business-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-12 -->
