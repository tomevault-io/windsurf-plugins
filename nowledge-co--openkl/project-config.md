---
trigger: always_on
description: This guide helps AI agents integrate with OpenKL for knowledge management, memory distillation, and **reproducible reasoning through citations**.
---

# OpenKL Agent Integration Guide

This guide helps AI agents integrate with OpenKL for knowledge management, memory distillation, and **reproducible reasoning through citations**.

## 🎯 **Core Principles for AI Agents**

### **When to Use Each Component**

1. **Grounding Store** → Raw external material (papers, docs, transcripts)
2. **Memory** → Your distilled insights and facts
3. **Citations** → Provenance tracking for reproducible reasoning
4. **Distillation** → Knowledge synthesis with full traceability

### **Agent Decision Tree**

```text
New Information Encountered
├── Is it raw external material? → Grounding Store
├── Is it a distilled insight? → Memory + Citations
├── Need to synthesize knowledge? → Distillation + Citations
└── Need to reference sources? → Use Citations
```

---

## 🚀 **Essential Commands**

### **Search & Discovery**

```bash
# Search memories
uv run ok mem search "transformer attention" --json

# Search grounding store
uv run ok store search "BERT bidirectional" --json

# Hybrid search across both
uv run ok search "transformer architecture" --json
```

### **Citation Management**

```bash
# Create citations for key findings
# Using jq (preferred):
uv run ok search "transformer" --json | jq -r '.[] | .id' | head -3 | \
  xargs -I {} uv run ok cite make {} --retention-class durable --tags "transformer"

# Fallback without jq:
# uv run ok search "transformer" --json | grep -o '"id": "[^"]*"' | head -3 | cut -d'"' -f4 | \
#   xargs -I {} uv run ok cite make {} --retention-class durable --tags "transformer"

# List all citations
ok cite list

# Open a citation to see full context
ok cite open 43411d2b03b695e1#chunk0036

# Verify citation integrity
ok cite verify 43411d2b03b695e1#chunk0036
```

### **Memory Management**

```bash
# Add memory with topics and tags
uv run ok mem add "Your insight here" --tags "tag1,tag2" --topics "topic1,topic2"

# Update existing memory
uv run ok mem update m-20250919-abc123 "Updated content"

# Search memories
uv run ok mem search "query" --json

# Delete memory
uv run ok mem delete m-20250919-abc123
```

### **Distillation Workflow**

```bash
# 1. Search for relevant information
uv run ok search "transformer architecture" --json > search_results.json

# 2. Create citations for key findings
cat search_results.json | jq -r '.[] | .id' | head -3 | \
  xargs -I {} uv run ok cite make {} --retention-class durable --tags "transformer,architecture"

# 3. Get distillation prompt for your LLM
uv run ok distill get-prompt memory-synthesis

# 4. Use the prompt with your LLM to distill content
# (Your agent evaluates the prompt and generates distilled content)

# 5. Create memory from distilled content
uv run ok distill create "Your distilled content here" \
  "43411d2b03b695e1#chunk0036,831514a53d5ea432#chunk0000" \
  --tags "transformer,architecture" --topics "nlp"

# 6. Verify the memory was created with proper relationships
uv run ok mem search "transformer" --json

# 7. Check graph relationships (use specific relationship queries)
uv run ok graph cypher "MATCH (m:MemoryNote)-[r:DerivedFrom]->(c:Chunk) RETURN m.id, c.id" --json
```

---

## 🔧 **Advanced Agent Workflows**

### **1. REPRODUCIBLE REASONING: Citations in Action**

```bash
# Search for information and create citations
uv run ok search "transformer architecture" --json | jq -r '.[] | .id' | head -3 | \
  xargs -I {} uv run ok cite make {} --retention-class durable --tags "transformer,architecture"

# Generate response with proper citations
echo "Based on my knowledge base, here are the key insights about transformer architecture:

$(cat search_results.json | jq -r '.[] | "- \(.text) (okcite://\(.id))"')

These insights are grounded in the following sources:
$(ok cite list | grep "transformer\|architecture" | head -3)"
```

**Example Output:**

```
Based on my knowledge base, here are the key insights about transformer architecture:

- Transformers use self-attention mechanisms for parallel processing... (okcite://m-20250919-283cb946)
- Self-attention allows the model to attend to different positions... (okcite://m-20250919-375470ed)
- Multi-head attention allows the model to jointly attend... (okcite://m-20250919-d88b47ef)

These insights are grounded in the following sources:
│ 43411d2b03b695e1#ch… │ chunk  │ verified │ /home/ubuntu/.ok/st… │ 2025-09-19 │
│ 831514a53d5ea432#ch… │ chunk  │ verified │ /home/ubuntu/.ok/st… │ 2025-09-19 │
```

**Why This Matters:** Citations provide **reproducible provenance** - every claim can be traced back to its source!

### **2. CONTEXT WINDOW MANAGEMENT: Smart Citation Usage**

```bash
# Instead of including entire documents, agents can cite specific sections
echo "Based on my knowledge base, here are the key insights about transformer architecture:

$(uv run ok search "transformer" --json | jq -r '.[] | "- \(.text) (okcite://\(.id))"')

These insights are grounded in the following sources:
$(ok cite list | grep "transformer\|architecture" | head -3)"
```

**Why This Matters:** Citations enable **context-efficient responses** - agents can reference sources without overwhelming the context window!

### **3. KNOWLEDGE SYNTHESIS: Agent-Driven Distillation**

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nowledge-co/OpenKL](https://github.com/nowledge-co/OpenKL) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
