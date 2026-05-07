---
trigger: always_on
description: Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.
---

# Questionnaire Multiagent Application - GitHub Copilot Instructions

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Fixing issues

- If you're in VS Code and you're asked to fix issues, always make a branch first.
- When asked to interact with GitHub, use its CLI if needed.
- Use the existing tests wherever possible. Ask a single question first. Use test_mock_excel_processing_1_sheet.py for fast mock testing, or test_live_excel_processing_1_sheet.py if you need to test against live Azure.
- Only make a new test class if absolutely necessary.
- Prefer mock tests (`test_mock_*.py`) for development - they're faster and don't require Azure credentials.

### Bootstrap and Setup
- Install Python dependencies: `pip install -r requirements.txt` -- takes 2-3 minutes on first install. NEVER CANCEL. Set timeout to 5+ minutes.
- Install GUI support: `sudo apt-get update && sudo apt-get install -y python3-tk` -- takes 3-5 minutes. NEVER CANCEL. Set timeout to 10+ minutes.
- Install testing framework: `pip install pytest` -- takes 30 seconds.
- Install Playwright browsers (optional): `playwright install` -- takes 5-15 minutes and may fail due to download issues. This is OK to skip if it fails.

### Environment Configuration
- Create `.env` file in project root with Azure AI Foundry configuration:
```bash
AZURE_OPENAI_ENDPOINT=https://your-project.services.ai.azure.com/api/projects/your-project
AZURE_OPENAI_MODEL_DEPLOYMENT=gpt-4o-mini
BING_CONNECTION_ID=your-bing-connection-name
APPLICATIONINSIGHTS_CONNECTION_STRING=InstrumentationKey=your-key;IngestionEndpoint=https://your-region.in.applicationinsights.azure.com/
```
- Azure CLI authentication is REQUIRED: `az login` before running the application.
- NEVER commit `.env` files to version control (already in .gitignore).
- If you're in VS Code, there's already a virtual environment present at ./venv. Activate this venv before you do anything.
- If you're in VS Code, you can interact with GitHub if needed using the GitHub CLI.

### Running and Testing
- Run GUI application: `python3 question_answerer.py`
- Run CLI application: `python3 question_answerer.py --question "Your question here" --verbose`
- Run CLI application in mock mode: `python3 question_answerer.py --question "Your question here" --mock --verbose`
- Check CLI help: `python3 question_answerer.py --help`
- Run tests: `python3 -m pytest tests/test_resource_cleanup.py -v` -- takes 5-10 seconds.
- Run tests with unittest: `python3 -m unittest discover tests/ -v` (note: some tests have import issues due to missing main module)

#### Test Types
The repository now contains two types of tests:

**Mock Tests (No Azure credentials required):**
- `test_mock_azure_question.py` - Mock Azure questions using mock mode
- `test_mock_excel_processing.py` - Mock Excel processing tests
- `test_mock_excel_processing_1_sheet.py` - Mock Excel processing with single sheet
- Use `mock_mode=True` parameter or `--mock` CLI flag
- Fast execution, predictable results, work without internet/credentials
- Question Answerer returns mock answers with https://www.microsoft.com link
- Answer Checker always returns "VALID"
- Link Checker validates microsoft.com successfully

**Live Tests (Require Azure credentials and .env setup):**
- `test_live_azure_question.py` - Live Azure questions using real Azure OpenAI
- `test_live_excel_processing.py` - Live Excel processing tests
- `test_live_excel_processing_1_sheet.py` - Live Excel processing with single sheet
- Require valid Azure credentials in .env file and `az login`
- Slower execution, real API calls, variable results based on AI responses
- Use for integration testing and validating actual Azure functionality

## Validation

### Dependency Installation Validation
- `pip install -r requirements.txt` should install 50+ packages successfully
- Expected packages: azure-ai-projects, azure-identity, python-dotenv, openpyxl, pandas, playwright, aiohttp, azure-monitor-opentelemetry, opentelemetry-sdk
- Installation time: 2-3 minutes for fresh install, under 2 seconds if already installed

### Application Validation
- Without Azure credentials: Application should exit with error "AZURE_OPENAI_ENDPOINT not found in environment variables"
- With invalid credentials: Application should show Azure authentication warnings and fail gracefully
- GUI mode test: Application starts but will exit if no Azure credentials are configured
- CLI help test: `python3 question_answerer.py --help` should show usage and examples

### Test Validation
- `python3 -m pytest tests/test_resource_cleanup.py -v` should pass all 15 tests in under 1 second
- `python3 -m pytest tests/test_mock_azure_question.py -v` should pass all mock tests in under 1 second (no Azure credentials required)
- `python3 tests/test_mock_excel_processing_1_sheet.py` should pass mock Excel test in under 10 seconds (no Azure credentials required)
- Tests validate Azure AI Foundry resource cleanup and error handling

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mcasalaina/QuestionnaireAgent_v2](https://github.com/mcasalaina/QuestionnaireAgent_v2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
