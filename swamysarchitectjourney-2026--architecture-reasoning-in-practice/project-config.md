---
trigger: always_on
description: enables: ["../advanced-content/"]
---

# Content Rules (Architecture Reasoning in Practice)

**Version**: 2.0  
**Last Updated**: January 1, 2026  
**Priority**: MANDATORY - All content creation must follow these rules without exception

---

## 📋 Rule Applicability

**IMPORTANT**: These rules apply differently based on content type:

### Practice Content (`src/01_reasoning-foundations/`, `src/02_answer-structuring/`, `src/03_tradeoff-articulation/`, `src/04_role-perspectives/`, `src/05_evaluation-scenarios/`)
- ✅ **File naming**: Descriptive names (e.g., `problem-framing.md`, `cqrs-selective-application.md`) - **NO numbering required** for content files
- ✅ **Folder naming**: Folders use numbered prefixes (`01_reasoning-foundations/`, `02_answer-structuring/`) - **ALWAYS numbered**
- ✅ **Line limits**: Recommended ≤150 lines (split, don't trim)
- ✅ **YAML frontmatter**: Recommended for content files (all 5 metadata fields when content is added)
- ✅ **Zero-copy policy**: Applies (content must be transformative)
- ✅ **File references**: Must point to existing files

**Numbering Rules Summary**:
- **Folders**: Always use numbered prefixes (`01_`, `02_`, etc.) - **NEVER use `00_`**
- **Content files**: Use descriptive names without numbering (e.g., `decision-rationale-framing.md`)
- **Split files**: When splitting content, use `-part1`, `-part2` suffixes (e.g., `topic-part1.md`, `topic-part2.md`) - no numbered prefix on content files

### Resources (`src/resources/` directory)
- ✅ **File naming**: Logical names (`frameworks.md`, `reference-materials.md`, `tools.md`)
- ✅ **Numbering**: NOT required (reference materials)
- ⚠️ **YAML frontmatter**: NOT required

---

## 🚫 Zero-Copy Policy (Non-Negotiable)

**CRITICAL**: All content (case study documentation and educational content) must be transformative, not reformative.

❌ **NEVER** copy text verbatim from books, articles, websites, videos, or third-party materials  
❌ **NEVER** mirror a source's outline, section order, headings, or example sequence  
❌ **NEVER** use "light paraphrasing" — must transform completely  
✅ **ALWAYS** create diagrams in Mermaid-first style with ASCII fallback (never embed copyrighted figures)  
✅ **ALWAYS** write fresh, minimal code from first principles  
✅ Brief quotations allowed ONLY with quotation marks and source citation

---

## 🔄 Transformative Workflow (Required Every Time)

**Step-by-step process for creating original educational content**:

1. **Source Intake**: Skim for intent and big ideas; don't copy notes verbatim
2. **Concept Map**: Create fresh outline with different sectioning tailored to Architecture Reasoning learning
3. **Teach Differently**: Use new analogies, scenarios, examples, use-cases (avoid source examples)
4. **Produce Original Artifacts**: Explanations, diagrams (Mermaid with ASCII fallback), minimal examples
5. **Cross-Link**: Add references and connections across Architecture Reasoning thinking modes
6. **Similarity Audit**: Ensure no sentences/structures resemble source
7. **Zero-Copy Verification**: **MANDATORY** - Verify no verbatim text, especially in quotes and "Key Principle" sections
8. **Optional References**: Add "References/Inspired by" links (no copied phrasing)

**Goal**: Create transformative educational content, not just reformative. Entirely new presentation, examples, and explanations that teach the same concepts through original methods.

**⚠️ CRITICAL REMINDER**: Even "Key Principle" quotes and example structures must be completely transformed. Verbatim copying of ANY text from source material violates the zero-copy policy.

---

## ⏱️ 25-Minute Learning Segments

**APPLICABILITY**: This rule applies to practice content files (`src/01_reasoning-foundations/`, `src/02_answer-structuring/`, `src/03_tradeoff-articulation/`, `src/04_role-perspectives/`, `src/05_evaluation-scenarios/`).

**For Lab Documentation**: The 150-line limit is a **recommendation**, not a strict requirement. Lab files may exceed 150 lines if needed for comprehensive instructions.

✅ **Modular content** designed for focused 25-minute sessions  
✅ **Multi-Part Structure**: Complex topics split into Part 1, Part 2, ... Part N  
✅ **One-Shot Learning**: Each segment complete and actionable within time limit  
✅ **Target Length**: 150 lines of content maximum per response (educational content)

### ⚠️ CRITICAL: Splitting vs. Trimming Policy

**APPLICABILITY**: This rule applies to practice content files (`src/01_reasoning-foundations/`, `src/02_answer-structuring/`, `src/03_tradeoff-articulation/`, `src/04_role-perspectives/`, `src/05_evaluation-scenarios/`).

**For Lab Documentation**: If lab files exceed 150 lines, consider splitting for better organization, but it's not mandatory.

**MANDATORY APPROACH** (Educational Content Only): When content exceeds 150 lines, **ALWAYS SPLIT** into multiple parts. **NEVER TRIM** or condense content.

**Why Splitting is Required:**
- ✅ **Preserves ALL educational content** - No loss of examples, explanations, or concepts
- ✅ **Maintains learning value** - Each part remains complete and actionable
- ✅ **Better learning experience** - Learners get comprehensive coverage across parts
- ✅ **Follows 25-minute principle** - Each part fits within focused learning session

**Why Trimming is Prohibited:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SwamysArchitectJourney-2026/architecture-reasoning-in-practice](https://github.com/SwamysArchitectJourney-2026/architecture-reasoning-in-practice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
