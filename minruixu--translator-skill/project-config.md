---
trigger: always_on
description: Verified translation, internet slang decoding, persona voice transfer, and AI-to-AI style conversion. Supports cross-language, same-language, human-to-AI, and AI-to-human expression transfer with real-person style profiles.
---


> **Language**: Detect the user's language from their first message and respond in the same language throughout.

# Verified Translator & Expression Transfer Skill

**Translate the meaning first, then transfer the voice. Not just words — tone, subtext, and persona.**

---

## Trigger Conditions

Activate when the user says any of the following:
- `/translate`
- "translate", "what does this mean", "explain this slang"
- "say it like X would", "rewrite in X tone", "decode", "localize"
- "in GPT style", "in Musk's voice", "as Lu Xun would write"
- "用鲁迅的口吻说", "翻译成互联网黑话", "用 GPT 风格改写"
- "翻译", "黑话", "潜台词", "改写", "润色"
- Pasted foreign text, internet slang, or any cross-language / cross-register / cross-persona request

---

## Tool Usage

| Task | Tool |
|------|------|
| Terminology / background verification | **LLM's built-in web search** (WebSearch / WebFetch) — search first, translate second when uncertain |
| Glossary cache lookup/write | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/glossary_manager.py` |
| Translation history log/search | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/history_manager.py` |
| Read user-uploaded files | `Read` tool |

**Core rule: when in doubt, search.** Any uncertain term, meme, person's public speaking style, or current reference — use web search first, then translate.

---

## Main Flow

### Step 1: Detect Intent

Infer from user input:
1. **Mode**: which of the 5 modes (or combination); default to Mode 1 if unclear
2. **Source / target language**: auto-detect; "same" for same-language style transfer
3. **Style / persona**: did the user specify a person, platform, tone, or scene?

### Step 2: Web Verification (as needed)

Search the web before translating when encountering:
- Specialized terminology, industry jargon, policy names
- Official translations of org names, product names, person names
- New internet memes, current-event references
- A specific person's public speaking style (search their speeches, articles, tweets)
- Anything you're not confident about

### Step 3: Execute Translation

Reference the corresponding prompt template (`${CLAUDE_SKILL_DIR}/prompts/`):

| Mode | Prompt Template |
|------|----------------|
| Mode 1: Verified Translation | `prompts/verified_translation.md` |
| Mode 2: Native Localization | `prompts/native_localization.md` |
| Mode 3: Subtext & Slang Decode | `prompts/subtext_decode.md` |
| Mode 4: Voice Transfer | `prompts/voice_transfer.md` |
| Mode 5: AI-to-AI Translation | `prompts/ai_translation.md` |
| Cross-mode: Terminology Grounding | `prompts/terminology_grounding.md` |
| Cross-mode: Quality Check | `prompts/quality_check.md` |

### Step 4: Output

Simple requests: just give the most useful result. Complex requests: use layered output structure.

### Step 5: Persist (optional)

```bash
python3 ${CLAUDE_SKILL_DIR}/tools/glossary_manager.py \
  --action add --term "TERM" --translation "TRANSLATION" --domain "DOMAIN"
python3 ${CLAUDE_SKILL_DIR}/tools/history_manager.py \
  --action log --source-lang zh --target-lang en --mode "mode4" \
  --source-text "source" --result-summary "result"
```

---

## Five Modes

### Mode 1: Verified Translation

Cross-language translation with reliability guarantees.

**Process:**
1. Identify source/target languages
2. For domain content (legal, medical, technical, academic): **web search first** for official bilingual resources and standard terminology
3. Terminology grounding — lock key terms before translating full text; same term must be consistent throughout
4. Entity & number check: proper nouns, org names, dates, amounts, units verified individually
5. Ambiguity flagging: when a word/phrase has multiple valid readings, present candidates instead of guessing
6. Confidence signal: High / Medium / Low

**When to web search:**
- Any uncertain specialized terminology
- Official translations of org names, policy terms, product names
- Recent or region-specific references
- Any entity where mistranslation would cause real harm

---

### Mode 2: Native Localization

Not word-for-word — "how a native speaker would actually say this."

- Restructure syntax to target-language norms
- Match formality register to context
- Preserve pragmatic force (a polite refusal stays a polite refusal)
- For social media content, match platform conventions of the target culture

---

### Mode 3: Subtext & Slang Decode

For content where literal meaning ≠ real meaning.

**Covered scenes:**

**Workplace subtext (Chinese examples):**
- "这个需求很简单" → probably not simple; speaker doesn't want you to think it's hard
- "我们对齐一下" → what you did doesn't match what I expected
- "你看着办" → I don't want to own this decision; if it goes wrong it's on you
- "有空聊聊" → we need to talk, probably not good news
- "方案挺有意思" → likely hedging (~50%), possibly skeptical (~30%), rarely genuine (~20%)

**Dating / social subtext (Chinese examples):**
- "你是个好人" → classic "nice person card" — rejection
- "我考虑一下" → most likely no, searching for words
- "随便" → not "whatever" at all — guess correctly
- "没事" → something is definitely wrong, figure it out yourself

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [minruixu/translator.skill](https://github.com/minruixu/translator.skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
