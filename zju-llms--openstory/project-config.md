---
trigger: always_on
description: This file provides guidance to Claude Code when working in `examples/WorldKernel`.
---

# CLAUDE.md — WorldKernel

This file provides guidance to Claude Code when working in `examples/WorldKernel`.

---

## Project Purpose

WorldKernel is a **text-to-interactive-world generation system** built on top of Agent-Kernel.

用户输入一句自然语言（如"创建最后一部中的霍格沃茨魔法学院"），系统经过多阶段 pipeline 生成完整的世界内容，最终适配 Agent-Kernel 启动可交互的多智能体仿真。

---

## Scope Constraint

**Only modify files under `examples/WorldKernel/`.** Sibling examples、`packages/`、repo root 均为只读依赖。

Runtime target: **`agentkernel_distributed`**.

---

## Pipeline Overview

```
User NL input
    │
    ▼
Stage 1  — 理解与模版准备                    ← IMPLEMENTED
    │       intent_parser → world_type_classifier → generation_planner → ontology_selector
    │       Output: templates/<session_id>/ (configs, models, generated/)
    ▼
Stage 2  — 世界内容语义生成                  ← PARTIALLY IMPLEMENTED
    │       InitDAGRunner 按拓扑分波执行 generation tools
    │       LocationGenerationTool: generate → review → retry → validate (完整)
    │       CharacterGenerationTool / PathGraphTool / RelationGraphTool: 接口已声明, run() 未实现
    ▼
Stage 3  — 校验、修复与适配                  ← NOT YET
    │       Patch Validation → AK Adapter
    ▼
Stage 4  — Agent-Kernel 仿真                ← NOT YET
            Tick-based multi-agent simulation
```

---

## Directory Structure

```
examples/WorldKernel/
├── CLAUDE.md
├── pyproject.toml
├── .env.example                    ← WORLDKERNEL_API_KEY
│
├── src/worldkernel/
│   ├── server.py                   ← FastAPI: API routes + static frontend (mount at /)
│   ├── constraints.py              ← GenerationConstraints (max_locations/max_characters)
│   │
│   ├── stage1/                     ← Stage 1 pipeline
│   │   ├── pipeline.py             ← run_stage1(): orchestrates modules, saves files + codegen
│   │   ├── intent_parser.py        ← parse_intent(): 深度意图解析
│   │   ├── world_type_classifier.py← build_world_template(): 世界模版构建
│   │   ├── generation_planner.py   ← plan_generation(): 生成计划 + 约束截断
│   │   ├── ontology_selector.py    ← generate_templates(): 6类实体模版 (并行LLM)
│   │   ├── world_spec.py           ← SessionInfo model
│   │   ├── types.py                ← IntentResult, WorldTemplate, GenerationPlan, EntityTemplate...
│   │   └── prompts/                ← Stage1 prompt templates (.md)
│   │
│   ├── llm/                        ← LLM call layer (all stages share)
│   │   ├── client.py               ← init(), chat(), chat_json() + JSON extract/repair
│   │   └── config_loader.py
│   │
│   ├── architect/                  ← Stage 2 semantic generation
│   │   ├── __init__.py             ← Public API re-exports
│   │   ├── init/                   ← Stage1→Stage2 桥接: 加载 artifacts, 编译 context
│   │   │   ├── loader.py           ← InitInputLoader.from_session_root()
│   │   │   ├── compilers.py        ← ContractCompiler, ExecutionDAGCompiler, SeedResolver
│   │   │   ├── models.py           ← InitBuildContext, ExecutionDAG, ResolvedSeed, Stage1ArtifactBundle
│   │   │   └── pipeline.py         ← compile_stage1_init_context()
│   │   ├── registry/               ← Schema + Tool 注册表
│   │   │   ├── core.py             ← SchemaRegistry, ToolRegistry
│   │   │   └── schema_loader.py    ← 从 session models/ 动态加载 Pydantic 模型
│   │   ├── semantic/               ← 生成执行核心
│   │   │   ├── runner.py           ← InitDAGRunner (拓扑分波, asyncio.gather)
│   │   │   ├── state.py            ← SemanticGenerationState (result_store)
│   │   │   ├── storage.py          ← save_semantic_artifacts()
│   │   │   └── bundle.py / repository.py / models.py
│   │   └── tools/                  ← Stage2 生成工具
│   │       ├── base.py             ← BaseStage2Tool, Stage2ToolRequest/Result/Context
│   │       ├── generation.py       ← CharacterGenerationTool, PathGraphTool, RelationGraphTool (stubs)
│   │       ├── identity_allocator.py ← IdentityAllocator + IdentityRegistry (确定性ID预分配)
│   │       └── generators/
│   │           ├── base_generator.py ← 共享工具函数 (prompt构建, schema内省, 校验)
│   │           ├── location_generator.py ← LocationGenerationTool (generate→review→retry 完整实现)
│   │           └── prompts/        ← Stage2 生成/评审/重试 prompt templates
│   │
│   └── models/                     ← 共享 Pydantic models
│       └── agent_schema.py
│
├── configs/
│   ├── models.yaml                 ← LLM config (OpenAI-compatible)
│   ├── architect.yaml              ← generation_constraints (max_locations: 7, max_characters: 10)
│   ├── simulation.yaml
│   └── storage.yaml
│
├── templates/                      ← Output: <session_id>/ (Stage1+Stage2 产出)
│   └── <session_id>/
│       ├── generated/
│       │   ├── artifact_manifest.json   ← Stage2 入口索引
│       │   ├── world_template.json
│       │   ├── plan/                    ← ontology_hints, instance_seed_catalog, execution_plan, world_background
│       │   ├── templates/<entity>/      ← 各实体各维度原始 JSON
│       │   └── artifacts/               ← Stage2 生成结果持久化
│       ├── configs/<entity>/            ← YAML 维度定义 (agent, location, path, relation)
│       └── models/                      ← 自动生成的 Pydantic 模型 + schema_manifest.json
│
└── frontend/                       ← Static HTML/CSS/JS (Vite dev optional), served by FastAPI at /
    ├── index.html
    ├── app.js
    ├── style.css
    ├── vite.config.ts
    └── package.json
```

---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ZJU-LLMs/OpenStory](https://github.com/ZJU-LLMs/OpenStory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
