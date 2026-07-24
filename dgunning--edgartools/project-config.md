---
trigger: always_on
description: The `edgar.xbrl` package handles **XBRL (eXtensible Business Reporting Language)** parsing, processing, and rendering from SEC filings. This package is responsible for converting raw XBRL XML into structured financial statements and queryable facts.
---

# XBRL Package - AI Assistant Guide

## Package Overview
The `edgar.xbrl` package handles **XBRL (eXtensible Business Reporting Language)** parsing, processing, and rendering from SEC filings. This package is responsible for converting raw XBRL XML into structured financial statements and queryable facts.

## Critical Architecture Points

### Class Hierarchy
```
XBRL (Core parser & data container)
├── FactsView (Query interface for facts)
├── FactQuery (Fluent query builder)
├── Statement (Single financial statement)
└── RenderedStatement (Rich-formatted statement output)
```

### Core Components
1. **xbrl.py** - Main XBRL parser and data container
2. **facts.py** - Query interface and fact processing
3. **statements.py** - Financial statement abstraction
4. **rendering.py** - Rich table formatting and display
5. **periods.py** - Period selection and fiscal logic
6. **models.py** - Data models for XBRL structures

## Data Availability Checking Methods

### 1. **Direct Fact Counting**
```python
# Basic fact counting
xbrl = filing.xbrl()
total_facts = len(xbrl._facts)           # Raw facts count
total_contexts = len(xbrl.contexts)      # Context count
total_periods = len(xbrl.reporting_periods)  # Available periods
```

### 2. **FactsView Query Interface** (Primary Method)
```python
facts = xbrl.facts
# Query builder methods:
facts.query().by_concept("Revenue").by_period_key("duration_2024-01-01_2024-12-31").to_dataframe()
facts.query().by_statement_type("IncomeStatement").by_fiscal_year(2024).to_dataframe()
facts.query().by_label("Net Income", exact=True).limit(10).to_dataframe()
```

**Available Query Filters:**
- `by_concept(pattern, exact=False)` - Filter by XBRL concept name
- `by_label(pattern, exact=False)` - Filter by display label
- `by_period_key(period_key)` - Filter by specific period
- `by_period_type("instant"|"duration")` - Filter by period type
- `by_statement_type(statement)` - Filter by statement type
- `by_fiscal_year(year)` - Filter by fiscal year
- `by_fiscal_period("FY"|"Q1"|"Q2"|"Q3"|"Q4")` - Filter by fiscal period
- `by_dimension(dimension, value)` - Filter by dimensional data
- `by_value(filter_func)` - Filter by fact values
- `by_custom(filter_func)` - Custom filter functions

### 3. **Period-Specific Data Checking**
```python
# Check facts for specific periods
period_key = "duration_2024-01-01_2024-12-31"
period_facts = facts.query().by_period_key(period_key).to_dataframe()
fact_count = len(period_facts)

# Check essential concepts for a statement
income_facts = facts.query().by_statement_type("IncomeStatement").by_period_key(period_key).to_dataframe()
revenue_facts = facts.query().by_concept("Revenue").by_period_key(period_key).to_dataframe()
```

### 4. **Statement-Level Data Availability**
```python
# Check if statement has data
try:
    statement = xbrl.statements.income_statement()
    data_available = statement is not None
except StatementNotFound:
    data_available = False

# Get raw statement data for analysis
stmt_data = xbrl.get_statement("IncomeStatement")
line_items = len(stmt_data) if stmt_data else 0
```

### 5. **Stitching Query Interface** (Multi-filing)
```python
from edgar.xbrl.stitching import XBRLS
xbrls = XBRLS([xbrl1, xbrl2, xbrl3])
stitched_facts = xbrls.facts.query().by_concept("Revenue").to_dataframe()
```

## Period Selection Logic

### Current Implementation (`periods.py`)
The period selection uses sophisticated fiscal-aware logic:

**For Annual Reports (FY):**
- Filters periods by duration > 300 days (prevents quarterly data)
- Selects up to 3 fiscal years for trend analysis
- Respects fiscal year boundaries and dates

**For Quarterly Reports (Q1-Q4):**
- Attempts year-over-year quarterly comparison
- Includes YTD periods when available
- Uses intelligent duration-based classification

**Key Functions:**
- `determine_periods_to_display()` - Main period selection
- `get_period_views()` - Available period view options
- `filter_periods_by_type()` - Filter instant vs duration
- `sort_periods()` - Sort by date (newest first)

## Data Quality Assessment

### 1. **Fact Completeness**
```python
def check_essential_concepts(xbrl, statement_type, period_key):
    """Check if essential concepts are present for a period."""
    essential = {
        'IncomeStatement': ['Revenue', 'NetIncome', 'OperatingIncome'],
        'BalanceSheet': ['Assets', 'Liabilities', 'Equity'],
        'CashFlowStatement': ['OperatingCashFlow', 'InvestingCashFlow', 'FinancingCashFlow']
    }
    
    required = essential.get(statement_type, [])
    found = 0
    
    for concept in required:
        facts = xbrl.facts.query().by_concept(concept).by_period_key(period_key).to_dataframe()
        if len(facts) > 0:
            found += 1
    
    return found / len(required) if required else 0.0
```

### 2. **Period Data Density**
```python
def calculate_period_density(xbrl, period_key):
    """Calculate fact density for a specific period."""
    total_facts = len(xbrl.facts.query().by_period_key(period_key).to_dataframe())

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dgunning/edgartools](https://github.com/dgunning/edgartools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
