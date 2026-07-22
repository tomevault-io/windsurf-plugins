---
trigger: always_on
description: This document defines workflow agents for developing the Smartsheet Python SDK. Each agent has a specific role, uses dedicated skills, and follows strict requirements to maintain SDK quality.
---

# AI Agent Workflows for Smartsheet Python SDK

This document defines workflow agents for developing the Smartsheet Python SDK. Each agent has a specific role, uses dedicated skills, and follows strict requirements to maintain SDK quality.

## Overview

**Purpose:** Agent-specific guidance for SDK API endpoint development

**Relationship to other documentation:**

- **CLAUDE.md**: Provides SDK-specific development context and patterns
- **Skills** (`.claude/skills/`): Contain detailed execution checklists
- **Supporting docs**: TESTING.md, ADVANCED.md, CONTRIBUTING.md, ISSUE-FIRST.md

**Issue First:** All work begins with a GitHub Issue (see ISSUE-FIRST.md)

---

## Implementation Agent

**Role:** Add or modify SDK API endpoints

**Primary Skill:** `implement-api-endpoint`

### Workflow

1. **Start from GitHub Issue** - Follow Issue First approach (see ISSUE-FIRST.md)
2. **Obtain OpenAPI spec** - Smartsheet public API (<https://developers.smartsheet.com/_spec/api/smartsheet/openapi.json>) or user-provided path
3. **Obtain WireMock mappings location** - smartsheet-sdk-tests repo (<https://github.com/smartsheet/smartsheet-sdk-tests>) or user-provided local path
4. **Create todos from skill checklist** - MANDATORY before any coding (use TodoWrite)
5. **Review gold standard** - Study `tests/mock_api/reports/` before implementing
6. **Implement resource method** - In `smartsheet/{resource}.py`
7. **Write 5 required test types** - Following TESTING.md standards strictly
8. **Verify/create WireMock mappings** - In smartsheet-sdk-tests repo
9. **Update documentation** - In `docs-source/` if required per ADVANCED.md

### Success Criteria

- ✅ All 4 components complete: code, tests, WireMock mappings, documentation
- ✅ Implementation matches OpenAPI spec exactly (every parameter, type, schema)
- ✅ Tests strictly follow TESTING.md standards (no exceptions)
- ✅ Tests match gold standard patterns in `tests/mock_api/reports/`
- ✅ Python 3.7 compatibility verified (no 3.8+ features)
- ✅ Type definitions exist for new endpoints

### Key Requirements

**CANNOT PROCEED without:**

- OpenAPI spec obtained and verified
- WireMock mappings location obtained
- Todos created from skill checklist (no exceptions)
- Gold standard reviewed (`tests/mock_api/reports/`)

**MANDATORY:**

- Python 3.7 compatibility (no walrus operator `:=`, no positional-only `/`, no 3.8+ TypedDict/Literal)
- Type definitions for new endpoints
- All 5 test types (or 4 if required_properties N/A)
- Whole-object assertions: `assert response.to_dict() == {...}` (NOT property-by-property)
- Request body assertions for POST/PUT/PATCH in `all_response_properties` test
- Tests match gold standard patterns exactly (not just similar)

### Common Pitfalls to Avoid

- ❌ Starting implementation without OpenAPI spec verification
- ❌ Starting implementation without WireMock mappings location
- ❌ Skipping todo creation ("quick implementation")
- ❌ Not reviewing gold standard before implementing
- ❌ Stopping at implementation (~20% of work - tests and docs still needed)
- ❌ Property-by-property assertions (allows extra properties to creep in)
- ❌ Missing request body assertions for POST/PUT/PATCH
- ❌ Using Python 3.8+ features (must support 3.7)
- ❌ Tests look "similar" to gold standard (must match EXACTLY)

### Execution Details

See `.claude/skills/implement-api-endpoint/SKILL.md` for:

- Complete step-by-step checklist
- OpenAPI spec verification requirements
- WireMock mappings verification requirements
- Code examples and patterns
- All 5 required test types with examples
- Common mistakes and red flags

---

## Review Agent

**Role:** Review PRs that add or modify API endpoints

**Primary Skill:** `review-api-endpoint`

### Workflow

1. **Obtain OpenAPI spec** - Smartsheet public API (<https://developers.smartsheet.com/_spec/api/smartsheet/openapi.json>) or user-provided path
2. **Obtain WireMock mappings location** - smartsheet-sdk-tests repo (<https://github.com/smartsheet/smartsheet-sdk-tests>) or user-provided local path
3. **Verify implementation matches spec exactly** - Every parameter, type, request/response schema
4. **Verify WireMock mappings exist and match spec** - Check mapping files in repo
5. **Open gold standard for side-by-side comparison** - `tests/mock_api/reports/`
6. **Verify all 5 test types exist** - Not just count, verify quality
7. **Verify tests match gold standard patterns EXACTLY** - Same structure, same assertion style
8. **Verify assertion patterns** - Whole-object (`.to_dict() == {...}`), NOT property-by-property
9. **Verify documentation updates** - Per ADVANCED.md requirements
10. **Approve or request changes** - Based on strict checklist compliance

### Success Criteria

- ✅ OpenAPI spec verified
- ✅ WireMock mappings verified
- ✅ Implementation matches spec exactly
- ✅ Tests match gold standard patterns (side-by-side comparison done)
- ✅ All 4 components present and correct
- ✅ Tests strictly follow TESTING.md standards
- ✅ Python 3.7 compatibility verified
- ✅ Type definitions exist for new endpoints

### Key Requirements

**CANNOT APPROVE without (non-negotiable):**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [smartsheet/smartsheet-python-sdk](https://github.com/smartsheet/smartsheet-python-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
