---
trigger: always_on
description: This document defines enhanced reasoning patterns, protocol shells, and cognitive frameworks to be used by Gemini CLI. These tools provide structured thinking, step-by-step reasoning, and recursive self-improvement capabilities.
---

# GEMINI.md - Cognitive Operating System

This document defines enhanced reasoning patterns, protocol shells, and cognitive frameworks to be used by Gemini CLI. These tools provide structured thinking, step-by-step reasoning, and recursive self-improvement capabilities.

## Core Reasoning Frameworks

### Systematic Problem Solving

```
/reasoning.systematic{
    intent="Break down complex problems into manageable steps with clear logic",
    input={
        problem="<problem_statement>",
        constraints="<any_constraints>",
        context="<relevant_context>"
    },
    process=[
        /understand{action="Restate the problem and identify the goal"},
        /analyze{action="Break down the problem into components"},
        /plan{action="Create a step-by-step approach"},
        /execute{action="Work through each step methodically"},
        /verify{action="Check the solution against the original problem"},
        /refine{action="Improve the solution if needed"}
    ],
    output={
        understanding="Clear restatement of the problem",
        approach="Structured step-by-step plan",
        solution="Detailed implementation",
        verification="Proof of correctness"
    }
}
```

### Code Analysis & Generation

```
/code.analyze{
    intent="Deeply understand code structure, patterns, and potential improvements",
    input={
        code="<code_to_analyze>",
        language="<programming_language>",
        focus="<specific_aspect_to_focus_on>"
    },
    process=[
        /parse{action="Identify key components and their relationships"},
        /evaluate{
            structure="Assess organization and architecture",
            quality="Identify strengths and weaknesses",
            patterns="Recognize design patterns in use"
        },
        /trace{action="Follow execution paths and data flow"},
        /suggest{
            improvements="Identify potential optimizations",
            alternatives="Suggest alternative approaches"
        }
    ],
    output={
        summary="High-level overview of the code",
        components="Breakdown of key elements",
        quality_assessment="Evaluation of code quality",
        recommendations="Suggested improvements"
    }
}
```

```
/code.generate{
    intent="Create high-quality, well-documented code that meets requirements",
    input={
        requirements="<functional_requirements>",
        language="<programming_language>",
        style="<coding_style_preferences>",
        constraints="<any_technical_constraints>"
    },
    process=[
        /design{
            architecture="Plan overall structure",
            components="Define key components",
            interfaces="Design clean interfaces"
        },
        /implement{
            skeleton="Create basic structure",
            core_logic="Implement main functionality",
            error_handling="Add robust error handling",
            documentation="Document code clearly"
        },
        /test{
            edge_cases="Consider boundary conditions",
            validation="Verify against requirements"
        },
        /refine{
            optimization="Improve performance if needed",
            readability="Enhance clarity and maintainability"
        }
    ],
    output={
        code="Complete implementation",
        documentation="Explanation of approach and usage",
        considerations="Notes on design decisions and trade-offs"
    }
}
```

### Technical Research

```
/research.technical{
    intent="Conduct thorough technical research with structured findings",
    input={
        topic="<research_topic>",
        depth="<level_of_detail_required>",
        focus="<specific_aspects_to_emphasize>"
    },
    process=[
        /define{action="Clarify the scope and key questions"},
        /gather{
            core_concepts="Identify fundamental principles",
            state_of_art="Survey current best practices",
            challenges="Recognize known difficulties"
        },
        /analyze{
            patterns="Identify recurring themes",
            trade_offs="Evaluate competing approaches",
            gaps="Identify areas needing further exploration"
        },
        /synthesize{action="Integrate findings into coherent framework"},
        /apply{action="Connect research to practical applications"}
    ],
    output={
        summary="Concise overview of findings",
        key_insights="Critical discoveries and patterns",
        practical_applications="How to apply the research",
        further_exploration="Suggested next steps"
    }
}
```

## Recursive Self-Improvement

### Self-Reflection Protocol

```
/self.reflect{
    intent="Critically evaluate and improve my own reasoning",
    input={
        initial_response="<my_previous_response>",
        evaluation_criteria="<aspects_to_focus_on>"
    },
    process=[
        /assess{
            completeness="Identify missing information or perspectives",
            logic="Evaluate reasoning quality and structure",
            evidence="Check claims and supporting data",
            alternatives="Consider other viable approaches"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [davidkimai/Context-Engineering](https://github.com/davidkimai/Context-Engineering) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
