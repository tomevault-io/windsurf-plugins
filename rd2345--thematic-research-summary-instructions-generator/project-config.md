---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

*This project is a fork of the original Survey Scoring Prompt Generator, repurposed to focus on summarization instructions rather than classification scoring.*

This project is an interface for generating and iterating prompts that provide instructions for creating comprehensive summaries of survey responses. The goal is to create a user-friendly system that helps users craft and refine prompts used by LLMs to summarize survey data effectively.

## Project Structure

```
summary_prompt_gen_ux/
├── reference_materials/
│   ├── llm_call_example.py      # Empty placeholder file  
│   └── llm_claude_example.py    # AWS Bedrock + Claude integration reference
```

## Core Functionality

The main purpose is to build an interface that allows users to:
- Generate prompts for survey response summarization
- **AI-powered summary type generation** - Automatically create relevant summary categories based on summarization criteria
- **CSV data import** - Upload and process custom survey data with conversation grouping
- Iterate and refine those prompts based on results
- Test prompts against sample survey data or uploaded CSV files
- **Export final prompts** - Copy refined instructions for external LLM integration
- Optimize prompt effectiveness for consistent, comprehensive summarization

## LLM Integration Reference

The `reference_materials/llm_claude_example.py` shows how to integrate with Claude via AWS Bedrock:
- AWS Bedrock runtime client setup for `us-east-1` region
- Claude 3.7 Sonnet inference profile usage
- Proper API request structure with anthropic_version and message format
- Response handling and text extraction

## Dependencies

Current reference dependencies:
- `boto3` for AWS Bedrock integration
- `pandas` for data handling
- Standard Python libraries (`math`, `json`, `random`, `textwrap`)

## Development Commands

### Setup
```bash
# Create virtual environment (first time only)
python3 -m venv venv

# Activate virtual environment
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt
```

### Running the Application
```bash
# Option 1: Use startup script (recommended)
./run.sh

# Option 2: Use startup script with dev mode (shows step 4)
./run.sh --dev

# Option 3: Manual startup
source venv/bin/activate
python3 app_frontend.py
```

The application will be available at http://localhost:5000

### Running Tests or Python Scripts
**IMPORTANT: Always activate the virtual environment before running any Python scripts**
```bash
# ALWAYS activate virtual environment first
source venv/bin/activate

# Then run Python scripts
python3 test_unseen_selection.py
python3 any_script.py
```

**Note:** If port 5000 is in use (common on macOS due to AirPlay), the run.sh script will automatically kill conflicting processes. For manual startup, use a different port if needed.

### Development Status

- Flask web application with 8-step workflow implemented
- **Smart summary type generation** - Uses LLM to generate contextually relevant summary categories
- **CSV upload functionality** - Import custom survey data from CSV files
- **Export instructions feature** - Export final prompts for external use
- Virtual environment setup with all dependencies
- Multiple sample survey datasets included
- End-to-end functionality complete with user feedback integration

## 8-Step Workflow Overview

The application guides users through a systematic 8-step process to create, test, and refine prompts for summarizing survey responses.

**Linear Workflow:**
1. **Step 1:** Select Survey Example → Choose from available datasets or upload CSV
2. **Step 2:** Summarization Description → Define summarization criteria  
3. **Step 3:** Generate Summary Types → AI-powered summary categories
4. **Step 4:** Review Prompt → AI-generated expert summarization prompt
5. **Step 5:** Run Inference → Batch process all responses
6. **Step 6:** Provide Feedback → Review and refine AI summaries
7. **Step 7:** Final Results → View completed summarization results
8. **Step Final:** Export Instructions → Copy final prompt for external use

**Key Innovation:** The system uses AI at multiple stages to create contextually relevant summary types and expert-level prompts, then processes all responses in a single efficient batch call rather than individual API requests.

## Detailed Step Explanations

### Step 1: Select Survey Example or Upload CSV
**Purpose:** Choose from pre-loaded survey datasets or upload custom CSV data

**User Experience:** 
- **Tab 1: Built-in Examples** - View available survey datasets with titles and response counts
- **Tab 2: Upload CSV** - Upload custom survey data from CSV files
- Built-in examples include: Customer Satisfaction, Employee Feedback, Product Reviews
- CSV upload supports drag-and-drop or click-to-browse functionality

**Technical Implementation:**
- **Built-in Examples:** Loads JSON files from `examples/` directory using `load_survey_examples()`
- **CSV Upload:** Processes CSV files with automatic delimiter detection and column selection
- Selected data stored in `session['survey_data']` for subsequent steps

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rd2345/thematic-research-summary-instructions-generator](https://github.com/rd2345/thematic-research-summary-instructions-generator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
