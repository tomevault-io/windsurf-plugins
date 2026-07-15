---
trigger: always_on
description: **Pacioli** is a personal finance tracking application named after Luca Pacioli, the father of accounting. It leverages Open Banking APIs (via GoCardless) to fetch transaction data and uses a hybrid approach (Regex + Agentic AI) to categorize spending.
---

# Pacioli Project Context

## Overview
**Pacioli** is a personal finance tracking application named after Luca Pacioli, the father of accounting. It leverages Open Banking APIs (via GoCardless) to fetch transaction data and uses a hybrid approach (Regex + Agentic AI) to categorize spending.

The core philosophy is **immutable raw data** combined with **derived state**. The raw JSON responses from the bank are stored permanently, and all categorization and analysis are computed on top of this ground truth.

## Architecture

### Data Pipeline
1.  **Ingestion (`update_transactions.py`):**
    *   Fetches data from GoCardless API.
    *   Writes **immutable** JSON files to `raw/<account_id>/<date>.json`.
    *   Uses an **overlapping fetch window** to ensure late-settling transactions are captured.
    *   **Idempotent:** Uses exclusive file creation (`x` mode) to prevent overwriting existing data.

2.  **Loading (`transaction_loader.py`):**
    *   Reads all JSON files from `raw/`.
    *   **Deduplicates** transactions based on `internalTransactionId`.
    *   Returns a flat list of unique `Transaction` objects.

3.  **Enrichment (`transaction_manager.py`):**
    *   Resolves categories using a priority hierarchy (`SOURCE_PRIORITY`):
        1.  **Manual Overrides** (`data/manual_assignments.json`)
        2.  **Transfer Matching** (paired inter-account transactions)
        3.  **Zero Amount Checks** (Ignored/Excluded)
        4.  **Regex Patterns** (`data/patterns.json`)
        5.  **Agent/AI Classification** (`data/llm_cache.json`)
    *   Outputs a flat **Polars DataFrame**.

## Data Schema & Constraints

### `data/patterns.json`
- **Format**: A dictionary where keys are the **Master Category List** and values are lists of pattern objects. 
- **Integrity**: This is the single source of truth for categorization. Every category used in the system **must** exist as a key here.
- **Validation**: Entries are validated with pydantic at load time (`PatternRule` in `transaction_manager.py`). A missing/empty/uncompilable `pattern`, unknown `field`, non-numeric bound, or unparseable time fails loudly with the offending category named — fix the entry rather than working around the error.
- **Fields (per object)**:
    - `pattern`: A regex string (applied case-insensitively).
    - `field`: (Optional) The transaction field to search. Defaults to `counterparty`. Valid values: `counterparty`, `remittance`, or `any` (both).
    - `clean_name`: The "human-friendly" merchant or entity name.
        - **Entity-First Philosophy**: This field MUST represent a specific entity (e.g., "Waitrose", "Uber", "British Gas"). Do NOT use groupings or categories (e.g., "Groceries", "Taxis", "Utility") as the name.
    - `min_amount`: (Optional) Minimum absolute amount to match.
    - `max_amount`: (Optional) Maximum absolute amount to match.
    - `min_day`: (Optional) Minimum day of month (1-31) to match.
    - `max_day`: (Optional) Maximum day of month (1-31) to match.
    - `min_time`: (Optional) Minimum time of day (ISO format, e.g., "11:30") to match.
    - `max_time`: (Optional) Maximum time of day (ISO format, e.g., "15:00") to match.

### `data/manual_assignments.json`
- **Format**: A dictionary mapping `internalTransactionId` to an object with `clean_name` and `category`.
- **Purpose**: Use for "one-off" transactions or outliers that don't warrant a recurring regex pattern.
- **Priority**: This is the **highest priority** source. It overrides Patterns, Transfers, and AI Cache.
- **Schema**:
    ```json
    {
      "tx_id": {
        "clean_name": "Merchant Name",
        "category": "Category > Subcategory"
      }
    }
    ```

### `data/llm_cache.json`
- **Format**: A dictionary caching classification results to avoid redundant research.
- **Integrity**: Entries can be updated by the agent using the `ops` skill. Mark decisions with `source: "AI_AGENT"`.

## Privacy & Security Guardrails

### 1. Data Classification
*   **Sensitive (Level 1)**: API Keys, Secret IDs, Tokens, and Personal Identifiers (`.env`, `token.json`, `TRANSFER_NAME`). **NEVER READ ALOUD OR COMMIT.**
*   **Personal (Level 2)**: Transaction history, raw JSON, CSV exports, account IDs, IBANs, real names. These live in `raw/`, `data/`, and `.csv` files.
*   **Configuration (Level 3)**: Regex patterns, category lists, logic. These are safe to share/commit *if* they don't contain hardcoded Level 2 data.

### 2. Pattern Design Mandates
*   **Disjoint Patterns**: All regex patterns in `data/patterns.json` MUST be disjoint (mutually exclusive) for any given transaction. 
  - The system picks the *first* matching pattern, which creates ambiguity if they overlap.
  - Use `min_amount`, `max_amount`, `min_time`, or `max_time` constraints to ensure that broad "catch-all" patterns do not overlap with specific "constrained" patterns.
  - Never rely on "fallback" behavior where a less-specific pattern is intended to catch misses from a more-specific one without explicit exclusion criteria in the broad pattern.

### 3. Standing Orders for AI Agents

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [genneth/pacioli](https://github.com/genneth/pacioli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-15 -->
