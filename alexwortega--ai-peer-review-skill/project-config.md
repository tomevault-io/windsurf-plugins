---
trigger: always_on
description: Use when the user asks for a peer review, critique, or meta-review of an academic paper (PDF, DOCX, or extracted text). Spawns N independent reviewer subagents in parallel under anonymized NATO codenames, then synthesizes a meta-review identifying common and unique concerns plus a CSV concerns table. Adapted from poldrack/ai-peer-review. Triggers on phrases like "peer review this paper", "critique this PDF", "review like a reviewer would", "meta-review this manuscript", "act as reviewer 2".
---


# Paper Peer Review

Multi-reviewer peer review of an academic paper. Models the workflow of [poldrack/ai-peer-review](https://github.com/poldrack/ai-peer-review) but uses parallel Claude subagents in place of multiple proprietary LLMs, so it works with no extra API keys.

## Inputs

| Argument | Required | Default | Notes |
|---|---|---|---|
| `paper` | yes | — | Path to a PDF, DOCX, or `.txt`/`.md` of the paper. |
| `domain` | no | inferred | Reviewer field, e.g. `"neuroscience and brain imaging"`. Inferred from the paper's title/abstract if not supplied. |
| `num_reviewers` | no | `3` | Independent reviewers to spawn. Min 3, max 8. |
| `output_dir` | no | `./papers/<paper-stem>/` | Where review artifacts are written. |
| `skip_meta` | no | `false` | If `true`, only individual reviews are produced. |
| `overwrite` | no | `false` | If `false`, reuse any `review_*.md` already present and only run missing reviewers + meta. |
| `alignment_critic` | no | `true` | If `true`, one of the `num_reviewers` slots is filled by an AI-Alignment-Forum-style critic (see `prompts/reviewer_alignment_forum.md`) instead of a generic reviewer. Set `false` to use only generic reviewers. |

If the user invokes the skill ambiguously, ask only for `paper` — infer the rest.

## Workflow

### 1. Extract paper text

- PDF → use `pypdf` (`python -c "from pypdf import PdfReader; ..."`) or `pdftotext` if available.
- DOCX → use `python-docx` or `pandoc`.
- `.txt`/`.md` → read directly.

If extraction yields fewer than ~1000 characters or text is mostly garbled (common with scanned PDFs), tell the user and stop — OCR is out of scope for this skill.

### 2. Sanity check

Confirm the document looks like an academic paper (abstract, methods/results, references). If not, ask the user to confirm before proceeding.

### 3. Spawn reviewers in parallel via `spawn_reviewers.py`

**Do NOT use the Agent tool to spawn reviewers.** In headless `claude -p` mode the host serializes tool_use blocks, which makes Agent-based parallelism collapse into sequential ~3 min/reviewer runs. We sidestep this by spawning reviewers as independent `claude -p` subprocesses through a Python helper.

Issue exactly **one Bash call**:

```bash
python3 {skill_dir}/scripts/spawn_reviewers.py \
  --paper-text-file /tmp/paper_text.txt \
  --output-dir <output_dir> \
  --skill-dir {skill_dir} \
  --num-reviewers <N> \
  --domain "<domain>" \
  --model sonnet \
  $([ <alignment_critic> = false ] && echo --no-alignment-critic) \
  $([ <overwrite> = true ] && echo --overwrite)
```

What this does:
- Forks `N` `claude -p --model sonnet` subprocesses concurrently. True OS-level parallelism — total wall-clock = `max(reviewer_time)`, not `sum(reviewer_time)`.
- Each child reads its prompt from stdin (paper_text + reviewer instructions), writes its review to `<output_dir>/review_<codename>.md` directly.
- Each child is a full Claude Code instance with tool access (Bash, arxiv_search, etc.) — no functional regression vs. Agent-based path.
- One slot is randomly assigned `prompts/reviewer_alignment_forum.md` if `alignment_critic=true`. Anonymity is preserved (the script does not disclose which slot was the critic).
- Skips reviewers whose `review_<codename>.md` already exists, unless `--overwrite` is passed (corresponds to skill's `overwrite=true`).

NATO codenames in order: `alfa, bravo, charlie, delta, echo, foxtrot, golf, hotel`. The script picks the first `N`.

The script writes progress lines to stderr:
- `[spawn_reviewers] alfa started (pid=…, t+0.0s)` — spawn event. Successive reviewers are staggered by 10 s so all 3 don't slam arXiv at once and trigger 429 cascades.
- `[spawn_reviewers] alfa OK (t+45.2s, size=12500)` — completed cleanly with sane content.
- `[spawn_reviewers] alfa FAIL_CONTENT (t+45.2s, size=87, has_verdict=False)` — claude -p exited 0 but produced an empty or unstructured output. Treat this as a failed reviewer; surface it to the user, don't silently accept.
- `[spawn_reviewers] alfa FAIL rc=N` — claude -p subprocess errored.

Parallelism check: all reviewer `started` lines must appear within ~10 × (N-1) seconds (e.g., N=3 → all 3 starts within 20 s). If they're minutes apart, something is wrong with the launcher.

Sonnet is mandatory for reviewers (`--model sonnet`). Don't downgrade to Haiku or upgrade to Opus without explicit user request — Sonnet 4.6 is the design point.

Reviewers must NOT see each other's outputs. The script enforces this by giving each subprocess its own stdin and not sharing state.

### 4. Save individual reviews

Write each subagent's return text to `<output_dir>/review_<nato>.md`. Keep the codename → (nothing, since they are all Claude) mapping trivial — the field exists in `results.json` for compatibility with the original tool.

### 5. Synthesize the meta-review


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AlexWortega/ai-peer-review-skill](https://github.com/AlexWortega/ai-peer-review-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
