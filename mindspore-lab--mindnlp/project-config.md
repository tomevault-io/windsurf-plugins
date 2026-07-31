---
trigger: always_on
description: MindNLP is a NLP/LLM library based on MindSpore, aiming to support HuggingFace Transformers and Diffusers on Ascend/GPU/CPU devices.
---

# MindNLP Project - Claude Code Configuration

## Project Overview

MindNLP is a NLP/LLM library based on MindSpore, aiming to support HuggingFace Transformers and Diffusers on Ascend/GPU/CPU devices.

## Directory Structure

```
mindnlp/
├── .claude/
│   ├── settings.json          # Permissions and hooks configuration
│   ├── agents/
│   │   ├── test-runner.md     # Test execution agent
│   │   ├── code-reviewer.md   # Code review agent
│   │   └── git-agent.md       # Git operations agent
│   ├── hooks/
│   │   ├── validate-command.sh    # Pre-execution command validation
│   │   └── post-edit-check.sh     # Post-edit code quality check
│   └── logs/
│       └── session-history.md # Historical session logs
├── src/
│   ├── mindnlp/               # MindNLP source code (editable)
│   ├── mindtorch/             # MindTorch v1 source code (editable)
│   └── mindtorch_v2/          # MindTorch v2 source code (editable)
├── tests/
│   ├── run_test.py            # Test runner (mindtorch v1)
│   ├── run_test_v2.py         # Test runner (mindtorch v2, uses torch_proxy)
│   └── transformers/          # HuggingFace transformers tests (read-only)
│       └── tests/models/      # Model-specific tests
└── CLAUDE.md                  # This file
```

## Current Status (as of 2026-02-06)

### mindtorch v1
- Tested on A-class and Qwen model families
- Known limitations: Clone kernel, meta device, model loading issues
- PRs: #2392, #2393

### mindtorch_v2
| Model | Architecture | Pass Rate | Status |
|-------|-------------|-----------|--------|
| Albert | Encoder | 98.2% (54/55) | Production-ready |
| BERT | Encoder | 79.1% (110/139) | Good |
| GPT-2 | Decoder | 44.3% (62/140) | Functional (non-generation) |

**Remaining gaps** (priority order):
1. Text generation utilities (`generate()`, beam search, sampling)
2. Gradient checkpointing (`torch.utils.checkpoint`)
3. Model serialization (SafeTensors edge cases, tied weights)
4. Model offloading (CPU/disk)

> Full session history: `.claude/logs/session-history.md`

---

## Multi-Agent System

### Agent 1: Test Runner (`test-runner`)

**Purpose**: Execute tests, analyze failures, and fix bugs automatically.
**Location**: `.claude/agents/test-runner.md`

**Usage**:
```
Use the Task tool with subagent_type="general-purpose" and reference the test-runner agent instructions.

Example prompt:
"Following the test-runner agent guidelines in .claude/agents/test-runner.md,
run the test file tests/transformers/tests/models/bert/test_modeling_bert.py
and fix any failures."
```

**Workflow**:
1. Activate conda: `source ~/miniconda3/bin/activate mindnlp`
2. Run: `python tests/run_test.py -vs {test_file}` (v1) or `python tests/run_test_v2.py -vs {test_file}` (v2)
3. Analyze test output for failures
4. Locate bug source in `./src/mindnlp/`, `./src/mindtorch/`, or `./src/mindtorch_v2/`
5. Apply targeted fixes
6. Re-run tests to verify

### Agent 2: Code Reviewer (`code-reviewer`)

**Purpose**: Scan and analyze code for quality, security, and best practices.
**Location**: `.claude/agents/code-reviewer.md`

**Usage**:
```
Use the Task tool with subagent_type="general-purpose" and reference the code-reviewer agent instructions.

Example prompt:
"Following the code-reviewer agent guidelines in .claude/agents/code-reviewer.md,
review the changes in src/mindnlp/transformers/models/bert/modeling_bert.py"
```

### Agent 3: Git Agent (`git-agent`)

**Purpose**: Handle git operations including push to origin and pull from upstream.
**Location**: `.claude/agents/git-agent.md`

**Usage**:
```
Use the Task tool with subagent_type="general-purpose" and reference the git-agent instructions.

Example prompt:
"Following the git-agent guidelines in .claude/agents/git-agent.md,
push the current changes to origin and then pull latest from ms master."
```

---

## Important Constraints

### Core Design Principle: No Transformers-Specific Customization

**CRITICAL**: MindTorch (both v1 and v2) must remain a **general-purpose PyTorch compatibility layer**.

- **NEVER** add transformers-specific hacks, workarounds, or special cases to mindtorch code
- **NEVER** check for `transformers` or model-specific classes in mindtorch implementations
- All fixes must be generic PyTorch API implementations, not transformers accommodations
- If a test fails due to transformers-specific behavior, document it as "not supported" rather than adding special cases

### Core Design Principle: mindtorch_v2 is Fully Independent

**CRITICAL**: mindtorch_v2 is completely independent of MindSpore and PyTorch. It must **NEVER** import or depend on either.

- **NEVER** import `mindspore`, `mindspore.ops`, `mindspore.mint`, or any mindspore submodule in mindtorch_v2 code
- **NEVER** import `torch` or any PyTorch module in mindtorch_v2 code
- All computation must be implemented via the internal dispatch mechanism and ACLNN/ACL C bindings (ctypes)
- The only external dependencies allowed are: `numpy`, `ctypes`, and the Python standard library

### For Test Runner Agent
- Only modify files in `./src/mindnlp/`, `./src/mindtorch/`, or `./src/mindtorch_v2/`
- **NEVER** modify test files in `./tests/transformers/`
- Always re-run tests after fixes


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mindspore-lab/mindnlp](https://github.com/mindspore-lab/mindnlp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
