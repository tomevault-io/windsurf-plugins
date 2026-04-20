---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ALYF Banking is a Frappe/ERPNext app that integrates bank accounts with ERPNext through EBICS (Electronic Banking Internet Communication Standard) protocol. It provides automated transaction synchronization, advanced bank reconciliation, and SEPA payment order processing for German, Austrian, French, and Swiss banks.

## Development Commands

### Running Tests
```bash
# Run all tests
bench --site [sitename] run-tests --app banking

# Run specific test file
bench --site [sitename] run-tests --app banking --module banking.ebics.doctype.sepa_payment_order.test_sepa_payment_order

# Run with coverage
bench --site [sitename] run-tests --app banking --coverage
```

### Code Quality
```bash
# Format Python code (uses ruff)
ruff format .

# Lint Python code
ruff check . --fix

# Format JavaScript
npx prettier --write "banking/**/*.js"

# Lint JavaScript
npx eslint "banking/**/*.js" --fix

# Run pre-commit hooks manually
pre-commit run --all-files
```

### Building
```bash
# Build assets (from bench root)
bench build --app banking

# Clear cache and rebuild
bench clear-cache && bench build --app banking
```

### Installation
```bash
# Install app on a site
bench --site [sitename] install-app banking

# Uninstall (safely removes all customizations)
bench --site [sitename] uninstall-app banking
```

## Architecture Overview

### Module Structure

**`banking/ebics/`** - EBICS protocol integration
- Manages direct bank connections via EBICS protocol (H004/H005)
- Downloads CAMT documents (camt.052 intraday, camt.053 statements, camt.054 batches)
- Key files:
  - `manager.py`: EBICS session management using fintech library
  - `utils.py`: Transaction sync, CAMT parsing, Bank Transaction creation
  - `doctype/ebics_user/`: EBICS credentials (keyring, passphrase, bank connection)
  - `doctype/ebics_request/`: Audit log of EBICS downloads
  - `doctype/sepa_payment_order/`: Outgoing payment batch management
  - `doctype/sepa_payment/`: Individual payment records within orders

**`banking/klarna_kosma_integration/`** - ALYF Banking Admin portal integration
- Communicates with ALYF's backend for license validation and user registration
- Key files:
  - `admin.py`: HTTP client for Banking Admin API
  - `doctype/banking_settings/`: Central configuration (API tokens, feature flags)
  - `doctype/bank_reconciliation_tool_beta/`: Advanced transaction matching
    - `bank_reconciliation_tool_beta.py`: Main DocType
    - `utils.py`: SQL query builders for flexible matching
    - `unpaid_vouchers.py`: Payment Entry/Journal Entry creation

**`banking/overrides/`** - ERPNext DocType customizations
- `bank_transaction.py`: Enhanced reconciliation validation (CustomBankTransaction)
- `bank_account.py`: IBAN validation hooks

**`banking/connectors/`** - External service communication
- `admin_request.py`: Generic HTTP client for Admin API
- `admin_transaction.py`: Transaction data parsing

**`banking/custom/`** - Custom fields and hooks for ERPNext DocTypes
- `purchase_invoice.py`: SEPA payment order creation, discount calculations
- `bank.js`, `employee.js`, `supplier.js`: Frontend customizations

### Key DocTypes and Relationships

```
EBICS Integration Flow:
EBICS User → EBICS Request → Bank Transaction → Bank Reconciliation Tool Beta

Outgoing Payment Flow:
Purchase Invoice → SEPA Payment Order → SEPA Payment (child) → EBICS Upload

Configuration:
Banking Settings (singleton)
  ├─ Banking Reference Mapping (custom field mappings per DocType)
  └─ Voucher Matching Default (default DocTypes for reconciliation)
```

### Bank Reconciliation Matching

The reconciliation tool uses a scoring system (0-7 points) to rank potential matches:
- Amount match: 1 point
- Date match: 1 point
- Reference number match: 1 point
- Description contains document number: 1+ points
- Party validation: filters non-matching parties

Custom fields can be added via Banking Settings → Banking Reference Mapping to match on additional fields like PO numbers or custom references.

### Transaction Sync

**Scheduled Jobs** (see `hooks.py`):
- Intraday sync: Every hour 7 AM - 6 PM on weekdays (`intraday_sync_ebics`)
- Full sync: Every 4 hours (`sync_all_accounts_and_transactions`)

**Sync Process**:
1. Retrieve EBICS Users with credentials
2. Initialize EBICS Manager with keyring
3. Download CAMT documents (C52/C53/C54)
4. Parse SEPA transactions from XML
5. Create Bank Transactions in ERPNext
6. Handle batch transaction splitting (uses subtransaction_id)

### SEPA Payment Orders

**Flow**:
1. Create from Purchase Invoice (button or bulk action)
2. Auto-populates recipient bank account and outstanding amount
3. Supports early payment discounts via `get_sepa_payment_amount()` hook
4. Status: Draft → Approved → Transmitted
5. Transmission options:
   - Download XML (manual upload to bank)
   - Send via EBICS (automated upload)

**Discount Handling**: If execution_date ≤ discount_date on Payment Schedule, discounted amount is used automatically.

## Important Patterns

### Frappe Whitelisting
Use `@frappe.whitelist()` for methods called from frontend. Example:
```python

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alyf-de) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
