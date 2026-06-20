---
trigger: always_on
description: Use when the user wants to turn economics PDF lecture slides or an economics course folder into portable Markdown revision notes, a problem-first 21-hour review roadmap, a refined existing review outline, or a current-scope past exam to lecture map. Trigger for: 总结课件, 复习资料, 考点总结, cheat sheet, 公式速查, summarize slides, review notes, lecture summary, review roadmap, 复习路线图, 复习排期, 21小时复习计划, problem-first revision plan, 润色大纲, 修改复习大纲, current-scope past exam map, past exam to lecture mapping, 今年考纲真题映射, 
---


# Econ Slides Review

Turn economics course materials into concise, exam-oriented portable Markdown notes. Optimize for three things: economic intuition, source traceability, and integration with problem patterns. Do not merely compress slides.

## Operating Rules

- Use a PDF-capable intake method for all PDFs. If a `pdf` or `pdf-reading` companion skill is available, use it for extraction and page handling.
- Default deliverable: write one `.md` file to disk in the course directory. Do not stop with a chat-only summary unless the user explicitly asks for chat-only output.
- Default language: Chinese-first, with important economics terms preserved in English parentheses, such as `边际替代率 (Marginal Rate of Substitution, MRS)`.
- Default scope: if the user points to a folder, scan the folder before summarising. If the user points to one specific PDF, treat it as the lecture source and produce a lecture-only review.
- Required citations: every knowledge point, formula, definition, model result, technique, and exam observation must include a precise source locator wrapped in Markdown inline-code backticks, such as `` `Lec 3 课件页 12-18` ``.
- Required intuition: every formal result, definition, formula, theorem, or model conclusion must be followed by a `#### Economic Intuition` subsection. If the intuition is unclear, write `#### Economic Intuition` followed by `直觉待补充` instead of silently omitting it.
- Never fabricate problem recommendations. Add `｜推荐练习：...` inside the same inline-code source tag only when a matching problem set, exam, seminar, or solution item was actually found.
- Do not solve homework or write essays. For problem sets and exams, extract tested concepts, question patterns, and answer techniques only.

## Mode Selection

Infer the mode before starting deep reading. Respect scope-limiting phrases such as "只生成", "只润色", "不要完整笔记", "roadmap-only", or "outline refinement".

| Mode | Trigger | Output |
|---|---|---|
| Full review | Default for summary/review-note requests | `<stem>_review.md` with integrated notes, past exam analysis when available, 21-hour roadmap, and quick reference |
| Roadmap only | `只生成 roadmap`, `复习排期`, `21 小时`, `problem-first roadmap`, `从题目出发安排复习` | `<stem>_roadmap.md`; do not generate full notes |
| Current-scope past exam map | `今年考纲真题映射`, `真题对应课件页`, `判断真题是否超纲`, `current-scope past exam map`, `past exam to lecture mapping` | `<stem>_current_scope_exam_map.md`; map every past exam subquestion to current scope, lecture location, and related problem sets |
| Outline refinement | `润色大纲`, `修改复习大纲`, `review outline`, `outline refinement`, or user points to an existing outline | Refined outline saved as `<outline-stem>_refined.md` unless the user asks to overwrite |

Mode rules:

- In full review mode, append a `21-Hour Problem-First Review Roadmap` section after the main topic content and past exam analysis. Keep `Quick Reference` as the final compact lookup section.
- In roadmap-only mode, scan the folder and build the concept-to-question map, but read lecture material only as needed to connect questions back to source pages and concepts.
- In current-scope past exam map mode, scan all past exam papers down to subquestion level, judge each subquestion against the current course scope, and keep all subquestions in the output with a scope status.
- In outline-refinement mode, do not regenerate the whole review note unless the user asks. Audit the existing outline against the course materials, propose changes, interact with the user on priorities, then write the refined outline.
- If the user requests multiple modes, combine them in the order requested.

## Folder Scan

Before reading content deeply, list the target course folder recursively up to 2 levels deep and classify available PDFs plus any existing outline/review Markdown or text files. Use subfolder names, filenames, and visible document clues.

| Category | Common patterns | Use |
|---|---|---|
| Syllabus / Outline | `syllabus`, `outline`, `course_outline`, `module_guide` | Topic order, assessment structure, exam format |
| Existing review outline | `review_outline`, `revision_outline`, `复习大纲`, `大纲`, `roadmap`, `review.md`, `outline.md` | Candidate for outline-refinement mode |
| Lecture slides | `lecture`, `lecture_notes`, `slides`, `课件`, `lec`, `week`, `topic` | Primary source, required for folder-level runs |
| Problem sets | `problem_set`, `pset`, `ps`, `homework`, `hw`, `exercises` | Tested concepts and question patterns |
| Seminar / Tutorial | `seminar`, `tutorial`, `习题课`, `section` | Extra intuition, worked examples, common mistakes |
| Solutions | `solution`, `solutions`, `sol`, `answers`, `mark_scheme` | Expected solution structure and techniques |
| Past exams | `past_exam`, `past_paper`, `exam`, `final`, `midterm`, `mock` | Exam format and recurring topic patterns |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AugustHuangMX/econ-review-master](https://github.com/AugustHuangMX/econ-review-master) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
