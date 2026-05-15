---
trigger: always_on
description: VIBE CHECK MCP ENGINEERING GUIDELINES
---

VIBE CHECK MCP ENGINEERING GUIDELINES

CRITICAL: Apply documentation-first rule to ALL third-party integrations.

STARTUP CHECKS:
Step 1: Verify Python environment with pip install -r requirements.txt
Step 2: Run PYTHONPATH=src python -m vibe_check server
Step 3: Execute pytest to verify all tests pass
Step 4: Check current version from VERSION file

ABSOLUTE RULES:
1. ALWAYS check official docs before custom solutions
2. ALWAYS use SDK over custom HTTP clients
3. ALWAYS test documented approach first
4. NEVER work on main branch directly
5. NEVER create files outside /tmp/ for temporary data
6. NEVER use oneOf/allOf/anyOf in MCP tool schemas
7. Files <700 lines, functions <40 lines
8. MANDATORY: Use vibe_check_mentor for ALL architecture decisions
9. MANDATORY: Use vibe_check_mentor when stuck for >30 minutes

THIRD-PARTY INTEGRATION WORKFLOW:
$$
Step 1: Check official documentation for standard approach
Step 2: Research existing solutions (GitHub, Stack Overflow)
Step 3: Create minimal POC with real data
Step 4: If complex, use mcp__clear-thought-server__mentalmodel
        Parameters: modelName="first_principles", problem="Is this complexity necessary?"
Step 5: If considering workaround, use mcp__clear-thought-server__decisionframework
        Parameters: decisionStatement="Work around bug vs use different API", analysisType="weighted-criteria"
Step 6: Build custom only if standard demonstrably fails
Step 7: Document WHY standard approach insufficient

RED FLAGS requiring immediate vibe_check_mentor:
- Building workarounds without checking docs
- Spending >30 minutes without progress
- Custom HTTP clients when SDKs exist
- Continuing due to sunk cost
- Any "should I build X vs Y" decision
- Integration/architecture planning

DEVELOPMENT WORKFLOW:

PHASE 1 - ISSUE CREATION:
Step 1: Search existing using GitHub MCP: "search issues in kesslerio/vibe-check-mcp for keywords"
Step 2: For complex problems, use mcp__clear-thought-server__sequentialthinking
        Parameters: thought="Define anti-pattern detection problem", thoughtNumber=1, totalThoughts=5
Step 3: Create issue using GitHub MCP: "create issue in kesslerio/vibe-check-mcp with title '[Type]: Description' body '<template>' labels 'P2,feature,area:pattern-detection'"

Required issue components:
- Title format: "[Type]: Clear description"
- Priority label: P0-P4 (MANDATORY)
- Type label: bug|feature|enhancement|documentation|test|refactor|security|performance|maintenance
- Area label: area:pattern-detection|area:educational-content|area:cli|area:mcp-integration|etc

PHASE 2 - IMPLEMENTATION:
Step 1: Create branch: git checkout -b feature/issue-{number}-{description}
Step 2: MANDATORY vibe_check_mentor consultation:
        vibe_check_mentor(query="Architecture approach for {feature}", reasoning_depth="standard")
Step 3: For architecture, use mcp__clear-thought-server__designpattern
        Parameters: patternName="modular_architecture", context="Anti-pattern detection system"
Step 4: For detection logic, use mcp__clear-thought-server__programmingparadigm
        Parameters: paradigmName="functional", problem="Pattern matching pipeline"
Step 5: Implement with FastMCP decorators and structured responses
Step 6: Keep functions <40 lines, files <700 lines

PHASE 3 - TESTING:
Step 1: Write tests in tests/ directory with test_*.py naming
Step 2: For debugging, use mcp__clear-thought-server__debuggingapproach
        Parameters: approachName="divide_conquer", issue="Isolating detection accuracy issue"
Step 3: Run: pytest --cov=src --cov-report=html
Step 4: Verify type hints: mypy src/

PHASE 4 - PR REVIEW:
Step 1: Create PR using GitHub MCP: "create PR from feature/issue-X to main"
Step 2: Check reviews: "show comments on PR X by kesslerio"
Step 3: For each review item, use mcp__clear-thought-server__decisionframework
        Parameters: decisionStatement="PR Review Item", options=["Fix now","Follow-up issue","Ignore"], analysisType="pros-cons"
Step 4: Categorize feedback:
        - CRITICAL ⚠️: Fix before merge
        - FOLLOW-UP 📋: Create issue if valuable
        - NICE-TO-HAVE 💡: Likely ignore (YAGNI)
        - OVERENGINEERING ❌: Reject complexity
Step 5: Update PR with critical fixes only
Step 6: Document changes: "add comment to PR X with summary of changes"
Step 7: Merge: "merge PR X with squash strategy"

ANTI-PATTERN DETECTION PATTERNS:

1. Infrastructure Without Implementation
   Detection: Custom solutions when standard APIs exist
   Example: Building HTTP client instead of using SDK

2. Symptom-Driven Development
   Detection: Fixing symptoms without root cause analysis
   Example: Adding workarounds instead of fixing core issue

3. Complexity Escalation
   Detection: Adding layers instead of simplifying
   Example: Abstract factory for simple function

4. Documentation Neglect
   Detection: Custom before checking official approaches
   Example: 2+ years building workaround for 3-line solution

VIBE CHECK MCP TOOLS:

Quick analysis during development:
mcp__vibe-check__analyze_github_issue(issue_number, repository="kesslerio/vibe-check-mcp", analysis_mode="quick")

Comprehensive analysis with GitHub posting:
mcp__vibe-check__analyze_github_issue(issue_number, analysis_mode="comprehensive", post_comment=true, detail_level="standard")

Text analysis:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kesslerio/vibe-check-mcp](https://github.com/kesslerio/vibe-check-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
