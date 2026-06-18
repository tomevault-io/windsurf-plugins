---
trigger: always_on
description: Validate and correct DOIs in academic references, check citation accuracy, verify reference lists, and identify fake or incorrect DOIs. Use this skill whenever the user mentions validating references, checking DOIs, correcting citations, verifying bibliographies, reviewing reference lists for publication, identifying citation errors, or needs to ensure reference accuracy before submitting manuscripts. Also trigger when users have reference files with potential DOI issues, need to verify DOIs in 
---


# Reference Validator

Validate and correct DOIs in academic reference lists. This skill helps researchers, students, and academics ensure their references are accurate and publication-ready.

## When to Use This Skill

Use this skill when:
- Validating DOIs in reference lists before manuscript submission
- Checking if references in a paper are real and correctly formatted
- Correcting incorrect DOIs in bibliographies
- Verifying citation accuracy for literature reviews
- Identifying fake or suspicious references
- Preparing reference lists for publication
- Checking reference quality for systematic reviews
- Validating DOIs in grant applications or theses

## Core Workflow

### 1. Assess the Input

First, examine the reference file:
- **Format**: Identify if it's markdown (.md), BibTeX (.bib), RIS (.ris), or plain text (.txt)
- **Structure**: Check how references are formatted (numbered, bulleted, etc.)
- **DOI format**: Look for DOI patterns (10.xxxx/xxxxx)
- **File size**: Large files (>100 references) may need batching

### 2. Extract and Validate DOIs

For each reference in the file:

1. **Extract the DOI** using regex pattern: `10\.\d{4,}/[^\s\]\'">\}]+`
2. **Clean the DOI** by removing trailing punctuation and whitespace
3. **Validate via Crossref API**:
   ```python
   import requests
   doi = "10.1038/nature12345"  # Example
   url = f"https://api.crossref.org/works/{doi}"
   response = requests.get(url, timeout=10)
   is_valid = response.status_code == 200
   ```

4. **Extract metadata** if valid:
   - Title
   - Journal name
   - Publication year
   - Authors

### 3. Identify Problematic DOIs

Flag DOIs that are:
- **Not found** in Crossref (404 response)
- **Annotation DOIs**: contain "annotation" in the DOI
- **Masthead DOIs**: contain "masthead" in the DOI
- **Preprints vs published**: Check if preprint DOI when published version exists

### 4. Correct Invalid DOIs (Optional)

For invalid or suspicious DOIs, attempt correction:

1. **Extract citation context** (title, authors, journal, year)
2. **Search Crossref** by title and first author
3. **Match with confidence scoring**:
   - Score > 90: High confidence (safe to correct)
   - Score 70-90: Medium confidence (flag for review)
   - Score < 70: Low confidence (keep original)

4. **Apply corrections** only for high-confidence matches

### 5. Generate Reports

Create a comprehensive validation report with:

**Summary Statistics:**
- Total references processed
- Valid DOIs (count and percentage)
- Invalid DOIs (count and percentage)
- Problematic DOIs requiring manual review

**Detailed Results:**
For each reference, include:
- Line number or reference ID
- Original DOI
- Validation status (valid/invalid/excluded)
- Corrected DOI (if applicable)
- Confidence score
- Title and journal (if found)

**Recommendations:**
- References needing immediate correction
- References requiring manual verification
- Suspicious patterns (e.g., multiple invalid DOIs from same journal)

## Output Format

### Corrected Reference File

Generate a new file with corrected DOIs while preserving the original format:

```markdown
# References

1. **Smith, J. et al. (2023).** Correct paper title. *Journal Name*, 12(3): 123-145. DOI: 10.1234/correct-doi

2. **Jones, A. et al. (2022).** Another paper. *Different Journal*, 45(2): 67-89. DOI: 10.5678/also-correct
```

### Validation Report

Create a detailed report (`validation_report.txt`) with:

```
====================================================================================================
DOI VALIDATION REPORT
====================================================================================================

SUMMARY:
Total references: 28
Valid DOIs: 24 (85.7%)
Invalid DOIs: 4 (14.3%)

====================================================================================================
INVALID REFERENCES REQUIRING ATTENTION:
====================================================================================================

Reference #4:
  Citation: Skarin, H. & Segerman, B. (2011). Horizontal gene transfer...
  DOI: 10.4161/mge.1.3.17714
  Status: NOT FOUND
  Action: Verify manually or remove from reference list

Reference #7:
  Citation: Smith, T. J. et al. (2021). The Distinctive Evolution...
  DOI: 10.3389/fmicb.2021.630537
  Status: ANNOTATION DOI
  Action: Replace with main paper DOI if available

====================================================================================================
CORRECTIONS APPLIED:
====================================================================================================

Reference #2:
  Original DOI: 10.1016/j.bbrc.2007.07.010
  Corrected DOI: 10.1016/j.bbrc.2007.06.166
  Confidence: High (score: 102.4)
  Action: Corrected automatically

[... more corrections ...]
```

## Best Practices

### Validation Strategy


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Tamoghna12/reference-validator-skill](https://github.com/Tamoghna12/reference-validator-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
