---
trigger: always_on
description: IMPORTANT: only follow these instructions if you were asked to add a new program/deal (content update request). Otherwise, ignore this file.
---

# Code Agent Instructions for cloudcredits.io

IMPORTANT: only follow these instructions if you were asked to add a new program/deal (content update request). Otherwise, ignore this file.

## How to add a new provider and program

When a user requests adding a new provider or program, follow this systematic process to replicate the n8n workflow functionality:

### Step 1: Extract Program URL from Issue
1. From the GitHub issue description, extract the program URL
2. This should be a single, valid URL to the startup program page

### Step 2: Scrape and Process Program Content
1. **Primary Method**: Use the clean text scraping script (`scripts/scrape-clean-text.js`) for comprehensive extraction
2. **Fallback Methods**: If the script fails or content is insufficient, use WebFetch or WebSearch tools
3. The script automatically cleans scraping residue (headers, footers, navigation elements)
4. Focus on extracting only meaningful program information about the deal, offer, or startup program
5. Preserve all important details without changing wording

#### Clean Text Scraping Script Usage
Use the dedicated script for robust and clean content extraction:

**Script Location**: `scripts/scrape-clean-text.js`
**Usage**: 
```bash
node scripts/scrape-clean-text.js <url>
```

**Example Usage**:
```bash
node scripts/scrape-clean-text.js https://example.com/startup-program
```

**Script Features**:
- Uses BrowserQL (Browserless' recommended approach) for text extraction
- Automatically removes common navigation and footer elements
- Normalizes whitespace and formatting
- Returns clean, structured JSON output
- Handles authentication using `BROWSERLESS_API_KEY` from environment variables available to you.

is**Output Format**:
```json
{
  "url": "https://example.com/startup-program",
  "status": 200,
  "title": "Program Title",
  "metaDescription": "Program description",
  "content": "Clean text content without scraping residue...",
  "success": true,
  "contentLength": 1234
}
```

**Error Handling and Fallback Strategy**:
1. **Primary**: Try the clean text scraping script first
2. **Fallback 1**: If script fails (API error, timeout, or insufficient content), use WebFetch tool
3. **Fallback 2**: If WebFetch provides insufficient details, use WebSearch tool to find additional information
4. **Content Validation**: Ensure extracted content includes:
   - Program benefits and features
   - Eligibility requirements
   - Application process details
   - Pricing/value information
   - Contact or support information

**Script Technical Details**:
- Uses Browserless `/unblock` endpoint for full HTML extraction
- Employs `html-to-text` npm package for professional text conversion
- Automatically handles authentication via query parameter
- Skips navigation, header, footer, and script elements
- Returns structured data with metadata
- Extracts content from full HTML including expanded elements

### Step 3: Enrich Program Data
Use this exact prompt to enhance the program information:

**Prompt**: "You will be given the initial information about the offer/deal/startup program below. Do a thorough online search and enrich this data from official source as much as possible. Especially focus on sections such as benefits, eligibility, FAQs. 

IMPORTANT: Only consider official sources from the providers of the deal. 

IMPORTANT: Return the full report without references."

**Input format**: 
```
---
INITIAL DATA ABOUT DEAL FROM {original_url}
---

{cleaned_scraped_content}
```

Focus on:
- Benefits and features
- Eligibility requirements and restrictions
- Application process details
- Frequently asked questions
- Program duration and timelines
- Contact information and support

### Step 4: Generate Structured Data
Generate TWO separate data structures using these exact prompts:

#### A. Program Data (YAML)
Use this exact prompt to generate the program JSON:

**Prompt**: "You are given the scraped text from the offer page containing information about cloud providers' programs, credits, discounts, etc. Your responsibility is to parse the content into the predefined JSON structure and return formatted JSON.

RULES:
1. Parse as much information as possible from the given website content. Don't drop anything.
2. Write about providers as third party. So refer to them/their, etc, instead of us, our, etc.
3. CRITICAL: Make sure you assign the most appropriate tags from the ones defined in src/tags.ts (ALLOWED_TAGS array).
4. Refer to the good examples of existing programs."

**Input format**:
```
---
INITIAL PROGRAM DATA
---

{cleaned_scraped_content}

---
ENRICHED PROGRAM DATA
---

{enriched_program_data}
```

Create a comprehensive program file following this structure:
```yaml
provider_slug: "provider-name"
title: "Program Name"
meta_title: "SEO-friendly title"
intro: "Brief compelling description (1-2 lines)"
description: "Detailed program description"
status: "Active"
url: "https://program-url.com"
min_value: 1000
max_value: 50000
value_type: "credits" # or "discount"
currency: "USD" # if applicable
tags: ["relevant", "tags"]
date: 2025-01-15

tiers:
  - name: "Tier Name"
    intro: "Tier description"
    max_value: 10000
    url: "https://tier-url.com"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [t3-sh/cloudcredits.io](https://github.com/t3-sh/cloudcredits.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
