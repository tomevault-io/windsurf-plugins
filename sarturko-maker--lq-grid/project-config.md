---
trigger: always_on
description: Bulk contract review for M&A lawyers. Upload contracts, define
---

# LQ Grid

Bulk contract review for M&A lawyers. Upload contracts, define
extraction columns, review results interactively, generate deliverables.

## Quick Start

```bash
# 1. Install Python deps
pip install -r requirements.txt

# 2. Install UI deps
cd src/ui && npm install && cd ../..

# 3. Start Claude Code with channel server
claude --dangerously-load-development-channels server:lq-ui-bridge

# 4. In another terminal, start the UI
cd src/ui && npm run dev
```

## Full Extraction Workflow

When pointed to a folder of contracts:

1. **Convert** documents to plain text:
   ```
   python3 src/pipeline/convert.py --input data/contracts/ --output data/output/texts/
   ```

2. **Copy originals** to UI public dir (for PDF/DOCX preview):
   ```
   mkdir -p src/ui/public/data/contracts
   cp data/contracts/*.pdf data/contracts/*.docx src/ui/public/data/contracts/
   ```

3. **Choose schema** — pick the right extraction template:
   - `templates/schemas/consent-review.json` — 11 columns (assignment, CoC, mechanism, action)
   - `templates/schemas/ma-dd-standard.json` — 14 columns (broader DD)
   - `templates/schemas/data-mapping.json` — 12 columns (GDPR)

4. **Extract** — spawn Sonnet reviewer agents to extract all schema columns.
   Each agent handles 5 documents and returns structured JSON with:
   - `value`, `source_quote` (verbatim), `source_location` (clause ref)
   - `source_start`, `source_end` (character offsets for highlighting)
   - `confidence`, `notes`

   **Parallelism**: launch up to 10 agents simultaneously in each wave.
   For 500 documents: 10 agents x 5 docs = 50 docs per wave, 10 waves total.
   Launch all agents in a wave using a single message with multiple Agent tool
   calls. Wait for all to complete, then launch the next wave.

   Write results to `data/output/results/batch-NNN.json`.

5. **Build manifest**:
   ```
   python3 src/pipeline/format_for_ui.py \
     --results data/output/results/ \
     --schema templates/schemas/consent-review.json \
     --output data/output/ui-manifest.json \
     --contracts data/contracts/
   cp data/output/ui-manifest.json src/ui/public/data/output/ui-manifest.json
   ```

6. The UI auto-refreshes every 3 seconds from the manifest.

## Architecture

Claude Code IS the extraction engine. No separate API key.
Claude Max subscription powers everything.

### UI ↔ Claude Code communication (Channels)

The React UI communicates with Claude Code via an MCP channel server.
The channel server (channel/ui-bridge.ts) runs on port 3002:
- Receives HTTP POSTs from the UI
- Pushes them into the CC session as `<channel>` events
- Exposes a `reply` tool for CC to send responses back
- Serves generated files (letters, reports) for download

### Data flow

1. Documents dropped into data/contracts/
2. convert.py → plain text in data/output/texts/
3. **Copy originals to src/ui/public/data/contracts/** (for PDF/DOCX preview)
4. Claude Code spawns Sonnet reviewer agents to extract structured data
5. Results written to data/output/results/ (one JSON per batch)
6. format_for_ui.py merges all batches → ui-manifest.json
7. Copy manifest to src/ui/public/data/output/ui-manifest.json
8. React UI auto-refreshes every 3 seconds

## Responding to channel events

When a `<channel source="lq-ui-bridge">` event arrives, parse the JSON.

### type: "upload" (Contracts uploaded via UI)

The user dropped contracts into the UI. The files are already saved to
data/contracts/ and src/ui/public/data/contracts/.

1. Run `python3 src/pipeline/convert.py --input data/contracts/ --output data/output/texts/`
2. Copy originals to `src/ui/public/data/contracts/` (upload handler does this automatically)
3. Spawn Sonnet reviewer agents — 10 parallel agents per wave, 5 docs each
4. After each wave, rebuild manifest and copy to UI so the grid populates progressively:
   ```
   python3 src/pipeline/format_for_ui.py --results data/output/results/ --schema templates/schemas/consent-review.json --output data/output/ui-manifest.json --contracts data/contracts/
   cp data/output/ui-manifest.json src/ui/public/data/output/ui-manifest.json
   ```
5. Continue waves until all documents are extracted
6. Reply confirming how many documents were processed

### type: "query" (Analyst chat)

1. Read data/output/ui-manifest.json
2. Answer as an **experienced M&A solicitor**
3. Use the **reply** tool with the request_id
4. Format in Markdown (tables need proper GFM syntax)
5. Use **Sonnet** model for queries

### type: "add_column"

1. Read the prompt and outputType from payload
2. Generate a short column title
3. Add to schema
4. Spawn Sonnet reviewers for ALL documents in data/output/texts/
5. Write results to data/output/results/batch-NNN-columnname.json
6. Run format_for_ui.py and copy manifest to UI public dir
7. Reply confirming completion

### type: "action"

1. Read the actionId from payload
2. For **consent-letters**: Read ACTUAL contracts, draft bespoke letters
3. For **reports**: Generate report, save as .md → convert to .docx
4. Reply confirming what was generated

## Source Highlighting

The crown jewel feature. When a user clicks "View Source" on a cell,
the original PDF or DOCX opens with the exact clause highlighted.

### How it works

**At extraction time (Sonnet):**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sarturko-maker/LQ-Grid](https://github.com/sarturko-maker/LQ-Grid) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
