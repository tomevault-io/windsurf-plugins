---
trigger: always_on
description: MeriNetWorth is a comprehensive net worth tracking system that consolidates financial data from multiple sources including bank accounts, equity holdings, mutual funds, fixed deposits, real estate, pension, and other assets. It provides visual analytics through a web dashboard.
---

## Project Overview

- @docs/

MeriNetWorth is a comprehensive net worth tracking system that consolidates financial data from multiple sources including bank accounts, equity holdings, mutual funds, fixed deposits, real estate, pension, and other assets. It provides visual analytics through a web dashboard.

**Supported Asset Types:**
- **Bank Accounts**: IDFC First, Equitas, Bandhan, ICICI, IndusInd, Kotak Mahindra
- **Equity Holdings**: CDSL and NSDL depositories
- **Fixed Income**: Term deposits/FDs from multiple banks
- **Real Estate**: Property valuations
- **Pension**: NPS, EPF, PPF accounts
- **Other Assets**: Cash, precious metals, etc.
- **Liabilities**: Loans, debts, receivables

**Critical Path**: Source Data → Python Parsers → JSON Output → Streamlit Dashboard

**IMPORTANT**: Never use `*Consolidated.xlsx` files in `data/` for analytics or processing. These files (e.g., `BankConsolidated.xlsx`, `EquityConsolidated.xlsx`, `MFConsolidated.xlsx`, `PensionConsolidated.xlsx`) are for personal use only and should not be parsed or integrated into the system.

## Development Commands

### Setup
```bash
pip install -r requirements.txt
```

### Data Processing
```bash
# Primary workflow: Launch Jupyter notebook
jupyter notebook notebooks/bank_data_processor.ipynb
# Then: Cell → Run All (or Shift+Enter per cell)
```

### Web Dashboard
```bash
# Quick launcher (recommended)
./run_dashboard.sh

# Manual launch
streamlit run web/app.py
# Opens at http://localhost:8501
```

## Architecture & Data Flow

### Two-Phase Architecture

**Phase 1: Data Extraction (Jupyter)**
- Input: Bank statements in `data/MM.YY/Bank/{BankName}/`
- Process: Bank-specific parsers extract account info
- Output: `output/bank_data.json` + `output/Bank-Consolidated-*.xlsx`

**Phase 2: Visualization (Streamlit)**
- Input: `output/bank_data.json`
- Process: Load, filter, and render interactive charts
- Output: Web dashboard at localhost:8501

### Parser Architecture

Each bank has a dedicated parser function in `src/bank_parsers.py`:
- Returns standardized dict: `{bank, account_number, holder_name, balance, source_file}`
- Parsers registered in `PARSERS` dict for programmatic access
- All parsers must handle errors gracefully and return `None` on failure

Each depository has a dedicated parser function in `src/equity_parsers.py`:
- Returns standardized dict: `{depository, dp_id, client_id, holder_name, total_value, holdings, source_file}`
- Parsers support: `parse_cdsl_statement()` and `parse_nsdl_statement()`
- All parsers must handle errors gracefully and return `None` on failure

**Parser Pattern**:
```python
def parse_<bank>_statement(file_path: Path) -> Optional[Dict]:
    # 1. Read file (pandas)
    # 2. Extract: account_number, holder_name, closing_balance
    # 3. Return standardized dict or None on error
```

**Equity Parser Pattern**:
```python
def parse_<depository>_statement(file_path: Path) -> Optional[Dict]:
    # 1. Read file (pandas - CSV for CDSL, Excel for NSDL)
    # 2. Extract: dp_id, client_id, holder_name, holdings list
    # 3. Calculate total_value from holdings
    # 4. Return standardized dict or None on error
```

### Key Data Structures

**Account Dictionary** (returned by all parsers):
```python
{
    'bank': str,           # Bank name
    'account_number': str, # Account identifier
    'holder_name': str,    # Account holder (may be empty)
    'balance': float,      # Current balance
    'source_file': str     # Source filename
}
```

**Equity Account Dictionary** (returned by equity parsers):
```python
{
    'depository': str,     # "CDSL" or "NSDL"
    'dp_id': str,          # Depository Participant ID
    'client_id': str,      # Client ID with depository
    'holder_name': str,    # Account holder name
    'total_value': float,  # Total portfolio value
    'total_holdings': int, # Number of holdings
    'holdings': [          # List of individual holdings
        {
            'isin': str,         # Security ISIN
            'name': str,         # Security name
            'quantity': float,   # Number of units
            'last_price': float, # Last closing price
            'value': float,      # Total value (price * quantity)
            'paid_up_value': float  # Paid up value per unit
        }
    ],
    'source_file': str     # Source filename
}
```

**JSON Output** (`output/bank_data.json`):
```python
{
    'generated_at': str,        # ISO datetime
    'total_balance': float,     # Sum of all balances
    'total_accounts': int,      # Account count
    'banks': dict,              # {bank_name: total_balance}
    'accounts': list[dict]      # List of account dicts
}
```

**Excel Output** (3 sheets):
1. "Raw Data": All account records
2. "Summary": Aggregated by bank
3. "Bank - Jun'25": FFS-compatible format

## Bank-Specific Parser Notes

### IDFC First Bank
- Format: Excel (.xlsx) with "Account Statement" sheet
- Extracts from header rows: ACCOUNT NUMBER, CUSTOMER NAME, Closing Balance
- Balance in 4th column (index 3) of "Closing Balance" row


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DivitMittal/MeriNetWorth](https://github.com/DivitMittal/MeriNetWorth) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
