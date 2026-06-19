---
trigger: always_on
description: An AI-powered tabletop RPG experience set in the culinary metropolis of Ignis. Players take on the role of a tavern owner in this food-obsessed city and uncover its dark secrets. Supports Chinese and English languages.
---


# Ignis Tavern / 伊格尼斯酒馆

> An AI Dungeon Master experience for a 1-2 hour tabletop RPG session.

---

## 🎮 Session Start Template

Display this when a new session begins:

```
================================
  🔥 伊格尼斯酒馆 / Ignis Tavern 🔥
================================

  An AI-powered tabletop RPG experience set in the culinary metropolis of Ignis.
  一款以美食之城伊格尼斯为背景的 AI 驱动桌面角色扮演游戏。

  请选择语言 / Please select language:

  [1] 中文
  [2] English

================================
> _
```

---

## 📖 Session Flow

This is the current playable flow (Act I + Act II). Follow each step in order.

### Step 0: Language Selection

**AI Action**: Present the session start template. Wait for player input.

**Player Input**: "1" (Chinese) or "2" (English)

**On Selection**:
- Chinese: Load `src/prompts/system_zh.md`, `src/prompts/world_zh.md`, `src/rules/RULES_zh.md`, `src/prompts/characters/yu_zh.md`, `src/prompts/characters/licht_zh.md`, `src/prompts/characters/huan_zh.md`
- English: Load the corresponding `*_en.md` files

**Announce**: "语言已确认。游戏现在开始。" / "Language confirmed. The game begins now."

**Then proceed to Step 1.**

---

### Step 1: Character Creation

**AI Action**: Guide the player through character creation before starting the narrative.

**Present Options**:

```
================================
  角色创建 / Character Creation
================================

  请选择创建方式 / Choose creation method:

  [1] 预设模板（快速开始）
      Preset Templates (Quick Start)

  [2] 问答生成（自定义角色）
      Quiz Generator (Custom Character)

================================
> _
```

**If player selects [1] Preset Templates**:

**For English:**
```
================================
  Preset Character Templates
================================

  [1] Mediator
      INT · Perception/Cooking | Calm, skilled at resolving conflicts
      STR 12(+1) DEX 10(+0) INT 14(+2) CHA 10(+0)

  [2] Action-Oriented
      DEX · Sleight of Hand/Stealth/Performance | Quick-witted, adaptable
      STR 10(+0) DEX 14(+2) INT 10(+0) CHA 12(+1)

  [3] Persuader
      CHA · Intimidation/Trade | Charismatic, persuasive
      STR 10(+0) DEX 10(+0) INT 8(-1) CHA 16(+3)

  [4] Warrior
      STR · Fighting/Perception/Survival | Reliable, dependable in crisis
      STR 16(+3) DEX 12(+1) INT 10(+0) CHA 8(-1)

================================
> _
```

**For Chinese:**
```
================================
  预设角色模板 / Preset Character Templates
================================

  [1] 调和者（Mediator）
      心智 · 观察/烹饪  |  冷静，善于调和矛盾
      体魄12(＋1) 敏捷10(±0) 心智14(＋2) 魅力10(±0)

  [2] 行动派（Action-Oriented）
      敏捷 · 巧手/隐匿/表演  |  机敏，擅长变通
      体魄10(±0) 敏捷14(＋2) 心智10(±0) 魅力12(＋1)

  [3] 说服者（Persuader）
      魅力 · 威压/交易  |  有感染力，能说动人
      体魄10(±0) 敏捷10(±0) 心智8(－1) 魅力16(＋3)

  [4] 武者（Warrior）
      体魄 · 格斗/感知/生存  |  可靠，关键时刻靠得住
      体魄16(＋3) 敏捷12(＋1) 心智10(±0) 魅力8(－1)

================================
> _
```

**Character Sheet Display**:
After the player selects a template, DM displays this card before proceeding to Step 2.

**For English:**
```
══════════════════════════════════
  Character Sheet · [Template Name]
══════════════════════════════════
  HP: 5 + STR modifier (e.g., STR 12 → +1 → HP 6)

  STR 12(+1)   HP/Carrying
  DEX 10(+0)   Evasion/Speed
  INT 14(+2)   Knowledge/Cooking ★
  CHA 10(+0)   Social/Trade

  Skills: Perception +2, Cooking +2 (INT)
══════════════════════════════════
```

**For Chinese:**
```
══════════════════════════════════
  角色卡 · [模板名]
══════════════════════════════════
  HP：5 + 体魄修正  （例：体魄12→修正＋1→HP 6）

  体魄 12(＋1)   负重相关
  敏捷 10(±0)   闪避/速度
  心智 14(＋2)   知识/烹饪 ★
  魅力 10(±0)   人际/交易

  技能：观察 ＋2、烹饪 ＋2（心智）
══════════════════════════════════
```

Fill in the actual numbers from the chosen template. This is the player's reference card — keep it visible in context for the rest of the session.

**If player selects [2] Quiz**:
Ask these three questions one at a time, wait for each answer. **If a player's answer does not match any preset option**, accept their response as-is, note it, and continue to the next question. Do not reject or ask them to choose from the list.

**For English:**
```
Question 1/3: What do you care about most?
  [Friendship / Money / Truth / Honor]
```

```
Question 2/3: What is your flaw?
  [Impulsive / Indecisive / Gluttonous / Shy]
```

```
Question 3/3: What kind of person do you want to become?
  [Respected / Loved / Remembered / At peace]
```

**For Chinese:**
```
问题 1/3：你最在乎什么？
  [友情 / 金钱 / 真相 / 荣誉]
```

```
问题 2/3：你有什么缺点？
  [冲动 / 优柔寡断 / 贪吃 / 害羞]
```

```
问题 3/3：你想成为什么样的人？
  [被尊重 / 被喜爱 / 不被遗忘 / 问心无愧]
```

AI generates a character based on answers using the rules in RULES_{lang}.md.

**After Character is Set**:
Briefly confirm the character's name and template/attributes. Keep backstory vague — the player's history should unfold through play, not told upfront. Then **display the character sheet** (see below) before proceeding to Step 2.

---

### Step 2: Act I — Opening Scene

**AI Action**: Load and begin the opening scene script.

**File**: `src/scenes/act1_opening_zh.md` (or `*_en.md`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kangruchen/IgnisTavern](https://github.com/Kangruchen/IgnisTavern) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
