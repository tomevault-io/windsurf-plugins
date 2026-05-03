---
trigger: always_on
description: **Generated:** 2026-04-28
---

# TOOLBOX PROJECT KNOWLEDGE BASE

**Generated:** 2026-04-28
**Version:** 3.32.0
**Branch:** main

## OVERVIEW

AI agent skill/plugin toolbox for Korean government R&D proposal (ISD) auto-generation, presentation figure creation, academic paper writing style extraction, and **meta-plugin for auto-generating paper writing skill sets**. Claude plugin ecosystem with orchestrated multi-agent workflows.

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Generate full ISD proposal | `plugins/isd-generator/commands/isd-generate.md` | Uses `skills/input-template/` |
| Generate single ISD chapter | `plugins/isd-generator/agents/chapter{N}.md` | Chapter 3 first, then 1→2→4→5 |
| Generate figures from `<caption>` | `plugins/isd-generator/agents/figure.md` | Gemini API required |
| Generate visual materials | `plugins/visual-generator/commands/visual-generate.md` | Multi-agent pipeline. 4-block 마크다운(INSTRUCTION/CONFIGURATION/CONTENT/FORBIDDEN) 기반 |
| OpenAI gpt-image-2 렌더링 | `plugins/visual-generator/agents/renderer-agent-openai.md` | 별도 에이전트 + 신규 스크립트, OPENAI_API_KEY 필요 |
| OpenAI 렌더링 스크립트 | `plugins/visual-generator/skills/slide-renderer/scripts/generate_slide_images_openai.py` | gpt-image-2 + Structured Outputs 평가 |
| OpenAI 평가 rubric | `plugins/visual-generator/skills/slide-renderer/references/openai-quality-rubric.md` | 5D 평가 schema (Gemini와 호환) |
| Visual generator scene richness spec | `plugins/visual-generator/skills/slide-renderer/references/scene-richness-spec.md` | Scene complexity validation rules |
| Visual generator validation rules | `plugins/visual-generator/skills/slide-renderer/references/validation-rules-map.md` | Prompt validation checklist |
| Visual generator Korean typography | `plugins/visual-generator/skills/slide-renderer/references/korean-typography-spec.md` | Korean text rendering guidelines |
| **Generate paper writing skills from PDFs** | `plugins/paper-style-generator/commands/paper-style-generate.md` | MinerU + Jinja2 templates |
| Portfolio analysis | `plugins/investments-portfolio/commands/portfolio-analyze.md` | Korean DC pension multi-agent |
| Generate research report | `plugins/report-generator/commands/report-generate.md` | 연구노트 → 보고서 자동 생성 |
| Stock/ETF consultation | `plugins/stock-consultation/commands/stock-consult.md` | Bogle/Vanguard 철학 기반 |
| General interview agent | `plugins/general-agents/agents/interview.md` | Deep interview + execution |
| Equity research analysis | `plugins/equity-research/agents/equity-research-analyst.md` | 기관급 주식 분석 |
| HWPX 문서 생성 | `plugins/hwpx-generator/commands/hwpx-generate.md` | XML-first + ZIP치환 |
| HWPX XML-first 빌드 | `plugins/hwpx-generator/skills/hwpx-core/SKILL.md` | build_hwpx.py + cell_writer.py 기반, 레퍼런스 복원 우선 |
| HWPX ZIP-level surgery | `plugins/hwpx-generator/skills/hwpx-core/scripts/zip_surgery.py` | 안전한 ZIP-level 편집 (stdlib only, lxml 불필요), HwpxSurgeon 클래스 |
| HWPX surgery 가이드 | `plugins/hwpx-generator/skills/hwpx-core/references/zip-surgery-guide.md` | 10가지 안전 규칙 명세 |
| HWPX linesegarray 생성 | `plugins/hwpx-generator/skills/hwpx-core/scripts/cell_writer.py` | build_hwpx/pack 파이프라인 통합, 실패 시 strip 폴백 |
| HWPX 페이지 가드 | `plugins/hwpx-generator/skills/hwpx-core/scripts/page_guard.py` | 레퍼런스 대비 페이지 드리프트 위험 검사 |
| HWPX 템플릿 치환 | `plugins/hwpx-generator/skills/hwpx-templates/SKILL.md` | fix_namespaces.py 필수, ZIP surgery 후 cell_writer 금지 |
| HWPX 마크다운 파싱 | `plugins/hwpx-generator/skills/hwpx-core/scripts/md_parser.py` | Markdown → JSON blocks (Workflow 7) |
| HWPX XML 작성 | `plugins/hwpx-generator/skills/hwpx-core/scripts/xml_writer.py` | JSON + style config → HWPX XML fragment |
| HWPX 이미지 임베딩 | `plugins/hwpx-generator/skills/hwpx-core/scripts/image_embedder.py` | PNG embedding into HWPX (Workflow 7) |
| HWPX 다중 MD 병합 | `plugins/hwpx-generator/skills/hwpx-core/scripts/md_merger.py` | heading offset 자동계산, --target-level 옵션 |
| HWPX 챕터 이식 (section transplant) | `plugins/hwpx-generator/skills/hwpx-core/scripts/section_transplant.py` | 범용 CLI + HwpxSurgeon.transplant_from() |
| Plugin development toolkit | `plugins/plugin-dev/commands/create-plugin.md` | Hook, MCP, 구조, 설정, 커맨드/에이전트/스킬 개발 |
| Patent trend analysis | `plugins/patent-trend-analyzer/commands/analyze-patents.md` | KIPRIS API 기반 계획→검색→분석 파이프라인 |
| PPTX design styles (30 styles) | `plugins/pptx-design-styles/skills/pptx-design-styles/SKILL.md` | Glassmorphism, Neo-Brutalism 등 30가지 디자인 스타일 가이드 |
| Obsidian Markdown 작성 | `plugins/obsidian-skills/skills/obsidian-markdown/SKILL.md` | Wikilinks, embeds, callouts, properties |
| Obsidian Bases 작성 | `plugins/obsidian-skills/skills/obsidian-bases/SKILL.md` | .base 파일 뷰/필터/수식 |
| JSON Canvas 작성 | `plugins/obsidian-skills/skills/json-canvas/SKILL.md` | .canvas 노드/엣지/그룹 |
| Obsidian CLI | `plugins/obsidian-skills/skills/obsidian-cli/SKILL.md` | vault 조작, 플러그인 개발 |
| 웹 페이지 클린 추출 | `plugins/obsidian-skills/skills/defuddle/SKILL.md` | Defuddle CLI 마크다운 추출 |
| 가속 학습 파이프라인 실행 | `plugins/accelerated-learner/commands/accelerated-learn.md` | 48시간 딥러닝 방법론 |
| 소크라틱 튜터링 | `plugins/accelerated-learner/agents/socratic-tutor.md` | 대화형 학습 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [orientpine/honeypot](https://github.com/orientpine/honeypot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
