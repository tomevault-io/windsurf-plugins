---
trigger: always_on
description: Toulmini is designed for moments of **contested logic** or **complex inquiry**. It is not for casual conversation.
---

# MCP Instructions & Agent Guide

## When to Use Toulmini

Toulmini is designed for moments of **contested logic** or **complex inquiry**. It is not for casual conversation.

**Use cases:**
-   **Debate Resolution**: When two parties disagree on a conclusion, use Toulmini to analyze the underlying logic of one or both sides.
-   **Fact-Checking & Analysis**: "Is [Claim] true?" -> Run the sequence to see if the evidence supports it logically.
-   **Self-Reflection**: "Is my belief about X rational?"
-   **Evaluating AI Outputs**: Use Toulmini to verify the reasoning of another LLM's output.

**Do NOT use for:**
-   Creative writing.
-   Simple Q&A ("What is the capital of France?").
-   Code generation.

## Tool Usage for Agents

Toulmini is a **sequential** toolset. Agents must follow the "Toulmin Loop" precisely. Do not skip steps.

### The Workflow

1.  **Receive Query**: User asks a reasoning question (e.g., "Is remote work productive?").
2.  **Phase 1**: Call `initiate_toulmin_sequence(query)`.
    *   *Action*: Execute the returned prompt.
    *   *Save*: `data_json` and `claim_json`.
3.  **Phase 2**: Call `inject_logic_bridge(query, data_json, claim_json)`.
    *   *Action*: Execute the returned prompt.
    *   *Save*: `warrant_json` and `backing_json`.
    *   *Check*: If the tool returns an error about weak logic, report that the argument failed.
4.  **Phase 3**: Call `stress_test_argument(query, data_json... backing_json)`.
    *   *Action*: Execute the returned prompt.
    *   *Save*: `rebuttal_json` and `qualifier_json`.
5.  **Phase 4**: Call `render_verdict(query, data_json... qualifier_json)`.
    *   *Action*: Execute the returned prompt.
    *   *Save*: `verdict_json`.
6.  **Phase 5 (Report)**: Call `format_analysis_report(...)` with all components.
    *   *Action*: Execute the returned prompt.
    *   *Output*: Present the final Markdown report to the user.

## When to Use the Council

The **Council of Experts** (`consult_field_experts`) is an optional but powerful feature for queries requiring specialized knowledge or diverse viewpoints.

### When to Convene the Council

**USE the Council when:**
-   **Ethical Dilemmas**: "Is genetic engineering morally acceptable?"
    -   Perspectives: `['Utilitarian Ethicist', 'Deontologist', 'Virtue Ethicist']`
-   **Scientific Claims**: "Is nuclear energy safe?"
    -   Perspectives: `['Environmental Scientist', 'Nuclear Physicist', 'Risk Analyst']`
-   **Legal Questions**: "Does this violate copyright?"
    -   Perspectives: `['IP Lawyer', 'Fair Use Scholar', 'Digital Rights Advocate']`
-   **Complex Policy**: "Should cities ban cars?"
    -   Perspectives: `['Urban Planner', 'Economist', 'Public Health Expert']`

**DO NOT use the Council for:**
-   Simple factual questions with clear answers
-   Queries where you already have sufficient expertise
-   Trivial or non-contested claims

### How to Integrate Council Output

**Council → Phase 2 (Backing):**
The `argument_for` from Council perspectives can strengthen your Backing in Phase 2.

**Council → Phase 3 (Rebuttals):**
The `argument_against` from Council perspectives can seed Rebuttals in Phase 3.

**Example Integration Pattern:**
1.  Query: "Is AI consciousness possible?"
2.  Convene Council: `['Neuroscientist', 'Philosopher of Mind', 'Computer Scientist']`
3.  Use Neuroscientist's `argument_for` → Enrich Phase 2 Backing
4.  Use Philosopher's `argument_against` → Seed Phase 3 Rebuttals

### Council Output Structure

```json
{
  "council_opinions": [
    {
      "perspective": "Neuroscientist",
      "argument_for": "Neural correlates suggest substrate independence...",
      "argument_against": "No evidence consciousness exists without biological neurons...",
      "key_citation": "Koch & Tononi, Neural Correlates of Consciousness (2008)"
    }
  ]
}
```

## Error Handling & Recovery

### Common Errors

**MISSING_COMPONENTS**
-   **Cause**: You forgot to pass output from a previous phase.
-   **Recovery**: Go back and retrieve the missing JSON component.

**TERMINATION_SIGNAL**
-   **Cause**: Warrant or Backing strength was "weak" or "irrelevant".
-   **Recovery**: This is a valid result. Report to the user:
    -   "The argument collapsed during Phase 2 because [reason from error]."
    -   Example: "WARRANT REJECTED: Strength is 'weak'. The logical bridge does not hold."

**QUERY_TOO_SHORT**
-   **Cause**: Query was less than 5 characters.
-   **Recovery**: Ask the user to elaborate on their question.

**INCOMPLETE_CHAIN**
-   **Cause**: Attempted to render verdict without all 6 components.
-   **Recovery**: Ensure you've completed Phases 1-3 before calling Phase 4.

### Circuit Breaker Behavior

Toulmini will **terminate immediately** if:
-   `Warrant.strength == "weak"` or `"irrelevant"` (Phase 2)
-   `Backing.strength == "weak"` or `"irrelevant"` (Phase 2)

When this happens:
1.  Do NOT attempt to continue to Phase 3
2.  Explain to the user that the argument failed logical validation
3.  Quote the specific error message from the TERMINATION_SIGNAL

## Complete Example Trace (with Council)

**Query:** "Is immortality desirable?"

### Step 1: Convene Council (Optional)
```
Call: consult_field_experts("Is immortality desirable?",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Hmbown/Toulmini](https://github.com/Hmbown/Toulmini) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
