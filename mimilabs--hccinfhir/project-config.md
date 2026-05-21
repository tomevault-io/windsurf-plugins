---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Information

**Current Version**: 0.1.5
**Package Name**: hccinfhir
**Description**: A Python library for calculating HCC (Hierarchical Condition Category) risk adjustment scores from healthcare claims data
**License**: Apache 2.0
**Python Requirements**: 3.9+

### What This Library Does

HCCInFHIR processes healthcare data to calculate Medicare risk adjustment scores used for payment calculations. It supports multiple input formats:

1. **FHIR ExplanationOfBenefit resources** - from CMS Blue Button 2.0 API
2. **X12 837 claim files** - from clearinghouses and encounter data
3. **X12 834 enrollment files** - for extracting dual eligibility and demographic data
4. **Direct diagnosis codes** - for quick calculations
5. **Service-level data** - standardized internal format

The library implements the official CMS-HCC risk adjustment methodology, including:
- Diagnosis code to HCC mapping
- Hierarchical condition category rules
- Demographic coefficients and interactions
- RAF (Risk Adjustment Factor) score calculations

## Development Commands

### Testing
```bash
hatch shell                    # Activate virtual environment
pip install -e .              # Install package in development mode
pytest tests/*                # Run all tests
pytest tests/test_filter.py   # Run specific test file
```

### Building and Publishing
```bash
hatch build                    # Build package
hatch publish                  # Publish to PyPI (maintainers only)
```

### Dependencies
- Install new dependencies by updating `pyproject.toml` dependencies
- Core dependency: `pydantic >= 2.10.3`
- Development dependency: `pytest`

## How Scripts and Modules Work

### Main Entry Points

#### 1. HCCInFHIR Class (`hccinfhir.py`)
The main processor class with three execution methods:

```python
from hccinfhir import HCCInFHIR, Demographics

# Initialize processor
processor = HCCInFHIR(
    filter_claims=True,  # Apply CMS filtering rules
    model_name="CMS-HCC Model V28",  # HCC model to use
    proc_filtering_filename="ra_eligible_cpt_hcpcs_2026.csv",  # CPT/HCPCS filtering
    dx_cc_mapping_filename="ra_dx_to_cc_2026.csv"  # Diagnosis mapping
)
```

**Method 1: `run()` - Process FHIR EOB Resources**
```python
# Input: List of FHIR ExplanationOfBenefit resources
eob_list = [{"resourceType": "ExplanationOfBenefit", ...}]
demographics = Demographics(age=67, sex="F")

result = processor.run(eob_list, demographics)
```

**Method 2: `run_from_service_data()` - Process Service-Level Data**
```python
# Input: Standardized ServiceLevelData objects
service_data = [ServiceLevelData(...)]
demographics = Demographics(age=67, sex="F")

result = processor.run_from_service_data(service_data, demographics)
```

**Method 3: `calculate_from_diagnosis()` - Direct Diagnosis Processing**
```python
# Input: List of diagnosis codes
diagnosis_codes = ["E11.9", "I10", "N18.3"]  # ICD-10 codes
demographics = Demographics(age=67, sex="F")

result = processor.calculate_from_diagnosis(diagnosis_codes, demographics)
```

#### Advanced: Demographic Prefix Override

All three methods support `prefix_override` parameter for cases where demographic auto-detection is incorrect:

```python
# ESRD patient with incorrect orec/crec codes (common data quality issue)
demographics = Demographics(age=65, sex="F", orec="0", crec="0")
diagnosis_codes = ["N18.6", "E11.22"]

# Force ESRD dialysis coefficients despite orec/crec being wrong
result = processor.calculate_from_diagnosis(
    diagnosis_codes,
    demographics,
    prefix_override='DI_'  # ESRD Dialysis prefix
)
```

**When to use prefix_override:**
- ESRD patients with orec='0'/crec='0' when they should be '2' or '3'
- Long-term institutionalized patients not properly flagged
- Dual-eligible status not correctly coded
- Any case where demographic categorization differs from administrative data

**Common prefix values:**
See "Coefficient Prefix Reference" section below for complete list.

### Core Processing Pipeline

#### Step 1: Data Extraction (`extractor.py`, `extractor_fhir.py`, `extractor_837.py`)

**Extract from FHIR:**
```python
from hccinfhir.extractor import extract_sld, extract_sld_list

# Single EOB
eob = {"resourceType": "ExplanationOfBenefit", ...}
service_data = extract_sld(eob)

# Multiple EOBs
eob_list = [eob1, eob2, eob3]
service_data_list = extract_sld_list(eob_list)
```

**Extract from X12 837:**
```python
from hccinfhir.extractor_837 import extract_sld_from_837

# X12 837 claim text
x12_text = "ISA*00*          *00*          *ZZ*..."
service_data = extract_sld_from_837(x12_text)
```

**Extract from X12 834 (Enrollment/Demographics):**
```python
from hccinfhir.extractor_834 import (
    extract_enrollment_834,
    enrollment_to_demographics,
    is_losing_medicaid,
    medicaid_status_summary
)

# X12 834 enrollment file
x12_text = "ISA*00*          *00*          *ZZ*..."
enrollments = extract_enrollment_834(x12_text)

# Get member demographics for risk calculation
for enrollment in enrollments:
    demographics = enrollment_to_demographics(enrollment)

    # Check if member is losing Medicaid coverage

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mimilabs/hccinfhir](https://github.com/mimilabs/hccinfhir) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
