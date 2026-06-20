---
trigger: always_on
description: Use when verifying academic citations for authenticity, checking if references are real or AI-generated, validating citation metadata, or detecting fabricated papers in manuscripts, grant proposals, or literature reviews
---


# Citation Verification

## Overview

**Systematic verification of academic citations against multiple authoritative databases to detect fabricated, incorrect, or AI-generated references.**

Core principle: Never give binary yes/no results. Always report match quality, evidence, and confidence scores.

## When to Use

Use this skill when:
- User asks to verify if citations are "real" or "AI-generated"
- Checking references in manuscripts, grants, or reviews
- Validating citation metadata (authors, year, journal, DOI)
- Detecting suspicious or fabricated papers
- User provides text with citations to verify
- User uploads a document and asks to check all references

Do NOT use for:
- Searching for new papers (use paper-ladder search clients)
- Generating citations (use citation-management skill)
- Formatting citations (use citation style guides)

## Extracting Citations from Documents

**When user provides a document (PDF, text, or pasted content):**

### Step 1: Extract References Section

Use regex or text parsing to find the references/bibliography section:

```python
import re

def extract_references_section(text: str) -> str:
    """Extract the references/bibliography section from document text."""
    # Common section headers
    patterns = [
        r'(?i)^#+\s*References\s*$',
        r'(?i)^References\s*$',
        r'(?i)^Bibliography\s*$',
        r'(?i)^Works Cited\s*$',
    ]

    for pattern in patterns:
        match = re.search(pattern, text, re.MULTILINE)
        if match:
            # Return everything after the header
            return text[match.end():]

    return text  # If no section found, process entire text
```

### Step 2: Parse Individual Citations

Extract structured citation data:

```python
def parse_citations(references_text: str) -> list[dict]:
    """Parse individual citations from references section."""
    citations = []

    # Split by common patterns (numbered lists, line breaks)
    lines = references_text.split('\n')

    for line in lines:
        line = line.strip()
        if not line or len(line) < 20:
            continue

        # Remove numbering: "1. ", "[1] ", etc.
        line = re.sub(r'^\[?\d+\]?\.?\s*', '', line)

        # Extract components using patterns
        citation = {}

        # Try to extract year
        year_match = re.search(r'\((\d{4})\)', line)
        if year_match:
            citation['year'] = year_match.group(1)

        # Try to extract title (usually in quotes)
        title_match = re.search(r'"([^"]+)"', line)
        if title_match:
            citation['title'] = title_match.group(1)

        # Try to extract DOI
        doi_match = re.search(r'doi:?\s*(10\.\d+/[^\s]+)', line, re.IGNORECASE)
        if doi_match:
            citation['doi'] = doi_match.group(1)

        # Try to extract arXiv ID
        arxiv_match = re.search(r'arXiv:(\d+\.\d+)', line)
        if arxiv_match:
            citation['arxiv'] = arxiv_match.group(1)

        citation['raw'] = line
        citations.append(citation)

    return citations
```

### Step 3: Verify Each Citation

Use paper-ladder to verify:

```python
from paper_ladder.clients import OpenAlexClient, CrossrefClient, SemanticScholarClient

async def verify_citations(citations: list[dict]) -> list[dict]:
    """Verify each citation against multiple databases."""
    results = []

    async with OpenAlexClient() as oa, \
               CrossrefClient() as cr, \
               SemanticScholarClient() as s2:

        for cite in citations:
            result = {'citation': cite, 'verification': {}}

            # Build search query
            if 'doi' in cite:
                # DOI is most reliable
                query = cite['doi']
            elif 'title' in cite:
                query = f'"{cite["title"]}"'
            else:
                # Use raw text as fallback
                query = cite['raw'][:100]  # Limit length

            # Search all databases
            try:
                result['verification']['openalex'] = await oa.search(query, limit=3)
                result['verification']['crossref'] = await cr.search(query, limit=3)
                result['verification']['s2'] = await s2.search(query, limit=3)
            except Exception as e:
                result['verification']['error'] = str(e)

            results.append(result)

    return results
```

### Complete Workflow Example

```python
import asyncio
from paper_ladder.extractors import PDFExtractor

async def verify_document_citations(file_path: str):
    """Complete workflow: extract PDF → parse citations → verify."""

    # 1. Extract text from PDF
    extractor = PDFExtractor()
    content = await extractor.extract(file_path)
    text = content.markdown

    # 2. Extract references section
    refs_section = extract_references_section(text)

    # 3. Parse individual citations
    citations = parse_citations(refs_section)

    print(f"Found {len(citations)} citations")

    # 4. Verify each citation
    results = await verify_citations(citations)

    # 5. Generate report (use structured format from protocol below)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SteadfastAsArt/citation-verification-skill](https://github.com/SteadfastAsArt/citation-verification-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
