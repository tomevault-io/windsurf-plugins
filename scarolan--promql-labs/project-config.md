---
trigger: always_on
description: PromQL Labs is a comprehensive educational resource for learning Prometheus Query Language (PromQL) through hands-on exercises. The project contains **twelve progressive labs** (0-11) organized into three difficulty levels (Beginner, Intermediate, Advanced), designed for workshop environments (particularly Killercoda).
---

# CLAUDE.md - AI Assistant Guide for PromQL Labs

## Project Overview

PromQL Labs is a comprehensive educational resource for learning Prometheus Query Language (PromQL) through hands-on exercises. The project contains **twelve progressive labs** (0-11) organized into three difficulty levels (Beginner, Intermediate, Advanced), designed for workshop environments (particularly Killercoda).

**Key Features:**
- 12 hands-on labs with progressive difficulty
- Reveal.js slide decks for each lab
- 100% automated test coverage for all queries
- CI/CD validation via GitHub Actions
- Real Node Exporter metrics, not synthetic data

## Repository Structure

```
promql-labs/
├── Beginner/           # Labs 0-2: Fundamentals, CPU exploration, CPU rates
├── Intermediate/       # Labs 3-4: Memory/filesystem, network/load
├── Advanced/           # Labs 5-11: Anomaly detection, correlation, rules,
│                       #            label manipulation, subqueries, histograms, joins
├── docs/               # Reveal.js slide decks (13 total: 1 overview + 12 labs)
│   ├── index.html      # Landing page with links to all decks
│   ├── common.css      # Shared slide styling
│   ├── common-scripts.js # Shared JavaScript (Grafana logo, speaker notes)
│   └── XX_LabName/     # Individual lab slide decks (index.html)
├── Tests/              # Query validation and coverage testing
│   ├── queries.py      # ALL test queries MUST be defined here
│   ├── test_queries.py # Main test runner (validates against live Prometheus)
│   ├── check_query_coverage.py # Ensures all lab queries have tests
│   ├── test_recording_rules.py # Validates recording rules
│   └── config.json     # Prometheus server configuration (USER MUST CONFIGURE)
├── Scripts/            # Helper scripts
│   ├── install-rules.sh # Sets up Prometheus recording/alerting rules
│   ├── histogram_traffic_generator.py # Generates histogram data for Lab 10
│   └── histogram_traffic_generator.sh # Bash version of traffic generator
└── .github/workflows/  # CI/CD for automated query testing
    └── promql-tests.yml # GitHub Actions workflow
```

## Key Conventions

### Lab Markdown Structure
Each lab follows this pattern:
1. **Title with emoji**: `# 🔍 Lab 0: PromQL Fundamentals`
2. **Objectives section**: Bullet points of learning goals
3. **Instructions section**: Numbered steps with queries and explanations
4. **Challenge section**: `<details>` spoiler tags for solutions
5. **Navigation link**: Link to next lab at bottom

### PromQL Code Blocks - CRITICAL
- **ALL PromQL queries MUST use triple backticks with `promql` language identifier**
- Lab queries use `instance="localhost:9100"` for Node Exporter
- Test queries use `$INSTANCE` placeholder (replaced at runtime with `localhost:9100`)
- Process exporter uses `instance="localhost:9256"`

Example:
```promql
rate(node_cpu_seconds_total{instance="localhost:9100",mode="user"}[5m])
```

### Slide Deck Standards - CRITICAL

**Framework:**
- Reveal.js 4.3.1 from jsdelivr CDN (NOT cdnjs or unpkg)
- Theme: `night` (dark background)
- Monokai syntax highlighting for code blocks
- Include `../common.css` and `../common-scripts.js`

**Title Slide Format (MUST FOLLOW EXACTLY):**
- Title uses `#` (h1), NOT `##`
- Format: `# Lab X: Title` (no emoji in title line)
- Emoji on subtitle line: `📊 Advanced PromQL`
- NO ALL CAPS anywhere
- Navigation link: `[All Slides](../index.html)`

Example:
```markdown
# Lab 8: Label Manipulation & Offset

📊 Advanced PromQL

[All Slides](../index.html)
```

**Common Mistakes to Avoid:**
- Using `##` for title (should be `#`)
- Putting emoji in title line (should be on subtitle line)
- Using ALL CAPS for emphasis
- Mixing HTML and Markdown inconsistently
- Using wrong CDN (must be jsdelivr)

### Testing Requirements - CRITICAL

**Core Rule: Every PromQL query in ANY lab markdown file MUST have a corresponding test in `Tests/queries.py`**

**Testing Workflow:**
1. Add query to lab markdown using ` ```promql ` code blocks
2. Add corresponding test to `Tests/queries.py` in appropriate `labX_queries` list
3. Run `python Tests/check_query_coverage.py` to verify coverage
4. Run `python Tests/test_queries.py` to validate queries work
5. CI will fail if any query is untested or produces an error

**Test Structure:**
```python
lab0_queries = [
    {
        "name": "Basic metric query",
        "query": "node_memory_MemTotal_bytes",
        "expected_type": "vector"  # or "matrix" or "scalar"
    }
]
```

## Python Environment Setup

**ALWAYS use a Python virtual environment:**

```bash
# Create and activate venv (first time)
python -m venv venv
source venv/bin/activate  # Linux/Mac
.\venv\Scripts\Activate   # Windows PowerShell

# Install dependencies
pip install requests
```

## Test Configuration

Before running tests or QA, update `Tests/config.json` with your Prometheus endpoint:

```json
{
    "prometheus_url": "http://localhost:9090/",
    "instance_name": "localhost:9100"
}
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [scarolan/promql-labs](https://github.com/scarolan/promql-labs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
