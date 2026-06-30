---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository shape

This is a documentation-heavy course repository for OpenClaw + Claude Code production practice. It is not one buildable application: each runnable lesson project has its own directory, virtualenv, runtime artifacts, and README.

Main areas:

- `openclaw-infra/`, `openclaw-im/`, `openclaw-models/`: deployment, IM integration, model/provider, security, and troubleshooting guides for OpenClaw.
- `openclaw-soul/`, `openclaw-heartbeat/`, `openclaw-skills/`: templates and examples for SOUL/AGENTS behavior files, Heartbeat/Cron automation, and SKILL.md authoring.
- `xhs-auto-publisher/`: Playwright-based Xiaohongshu publisher for authorized accounts, with human QR-login handoff and Lobster/Feishu notification payloads.
- `financial-automation/`: Python OCR/PDF extraction pipeline for invoices, validation, attachment upload, and Feishu Bitable write-back.
- `morning-newspaper/`: Python multi-source news collection pipeline with three LLM editorial gates, static dashboard generation, and Feishu delivery workflow.
- `CRM-Assistant/`: Python standard-library CLI that converts Feishu meeting raw data/transcripts into CRM assets and Feishu Bitable rows.
- `ai-quant-cli/`: Python CLI that parses A-share annual-report PDFs into structured financials; Claude Code itself performs the risk analysis (no LLM API in code), then scripts render charts and a single-page HTML investment report.
- `github-secret-auditor/`: OpenClaw Skill with no runnable code; it drives Claude Code over ACP to audit and remediate leaked secrets in an authorized GitHub repo, with OpenClaw handling review, commit/push, and the Feishu report.

No Cursor rules, `.cursorrules`, or GitHub Copilot instruction file are present at repository root.

## Common commands

There is no repo-wide build, lint, or test command. Work from the relevant subproject directory.

### Financial automation

```bash
cd financial-automation
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
python scripts/run_skill_job.py runtime/sample_run_input/hotel_invoice.pdf
```

Run the available smoke test module:

```bash
cd financial-automation
python -m unittest tests.test_smoke
```

Run one test class or one test method:

```bash
python -m unittest tests.test_smoke.OCRExtractSmokeTest
python -m unittest tests.test_smoke.OCRExtractSmokeTest.test_parse_rail_ticket_fields
```

The smoke tests create temporary files under `financial-automation/.tmp_tests/`.

### Morning Newspaper

```bash
cd morning-newspaper
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
```

Local collection checks:

```bash
python scripts/collect_raw.py --dry-run
python scripts/collect_raw.py --skip-tavily
python scripts/enrich_content.py
```

Batch pipeline entry points:

```bash
python scripts/run_daily_pipeline.py
python scripts/run_daily_pipeline.py --skip-tavily
python scripts/run_daily_pipeline.py --rebuild-dashboard-only
python scripts/check_runtime_status.py
```

The full Skill workflow is staged. The three LLM result files must be generated between prepare/apply steps, rather than relying on `run_daily_pipeline.py` to invent them:

```bash
python3 scripts/collect_mailbox.py
python3 scripts/collect_raw.py
python3 scripts/enrich_content.py
python3 scripts/prepare_title_shortlist.py
# produce runtime/title_shortlist_result.json from runtime/title_shortlist_prompt.txt
python3 scripts/apply_title_shortlist.py
python3 scripts/prepare_draft_input.py
# produce runtime/draft_result.json from runtime/draft_prompt.txt
python3 scripts/apply_draft_results.py
python3 scripts/prepare_top10_ranking.py
# produce runtime/top10_ranking_result.json from runtime/top10_ranking_prompt.txt
python3 scripts/apply_top10_ranking.py
python3 scripts/build_dashboard.py
python3 scripts/check_runtime_status.py
```

Dashboard/debug commands:

```bash
./scripts/serve_dashboard_8510.sh
streamlit run scripts/dashboard_app.py
```

### CRM Assistant

```bash
cd CRM-Assistant
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

`requirements.txt` is intentionally empty of third-party packages; the CLI currently uses the Python standard library.

Local validation without Feishu credentials:

```bash
python scripts/crm_assistant.py build-context-from-feishu \
  --raw-input-path assets/feishu_raw/pingan_longxiahezi_need_confirmation.json \
  --output-dir runtime/quick_start

python scripts/crm_assistant.py process-transcript \
  --transcript-path runtime/quick_start/transcript.txt \
  --context-path runtime/quick_start/context.json \
  --output-dir runtime/quick_start
```

Useful CLI checks:

```bash
python scripts/crm_assistant.py run-sample-tests
python scripts/crm_assistant.py run-feishu-pipeline-tests
python scripts/crm_assistant.py run-model-output-tests
```

Feishu write-back is available through `inspect-feishu-bitable`, `sync-feishu-bitable`, and `ingest-feishu-raw-to-bitable`; prefer `--dry-run` before real writes.

### XHS auto publisher

```bash
cd xhs-auto-publisher
pip install -r requirements.txt
python -m playwright install chromium

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DjangoPeng/agentic-ai](https://github.com/DjangoPeng/agentic-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-30 -->
