---
trigger: always_on
description: When using dsy, the LLM/AI framework
---

# DSPy: Programming Foundation Models

## What is DSPy?

DSPy is a framework for programming—rather than prompting—language models. It allows you to build modular AI systems and offers algorithms for optimizing their prompts and weights. Instead of brittle prompts, you write compositional Python code and use DSPy to teach your LM to deliver high-quality outputs.

**Key Value Proposition:** DSPy replaces manual prompt engineering with automated optimization, making AI systems more reliable, maintainable, and effective.

## Installation

```bash
# Install DSPy
pip install dspy

# Install from latest main branch
pip install git+https://github.com/stanfordnlp/dspy.git

# With optional dependencies
pip install dspy[anthropic,aws,mcp]  # For specific providers
```

## Core Concepts

### 1. Signatures
Define the input/output structure of your AI modules:

```python
import dspy

class TranslateSignature(dspy.Signature):
    """Translate text from one language to another."""
    text = dspy.InputField()
    source_language = dspy.InputField()
    target_language = dspy.InputField()
    translation = dspy.OutputField()

# Or use string syntax
translate_sig = dspy.Signature("text, source_language, target_language -> translation")
```

### 2. Modules
Building blocks that use language models:

```python
class Translator(dspy.Module):
    def __init__(self):
        super().__init__()
        self.translate = dspy.Predict(TranslateSignature)
    
    def forward(self, text, source_language, target_language):
        return self.translate(
            text=text,
            source_language=source_language,
            target_language=target_language
        ).translation
```

### 3. Language Model Configuration
Configure any supported LM provider:

```python
# OpenAI
lm = dspy.LM(model="openai/gpt-4o-mini")

# Anthropic Claude
lm = dspy.LM(model="anthropic/claude-3-5-sonnet-20241022")

# Local models via Ollama
lm = dspy.LM(model="ollama/llama3.2")

# Configure DSPy to use the model
dspy.settings.configure(lm=lm)
```

### 4. Built-in Modules
DSPy provides powerful pre-built modules:

```python
# Chain of Thought reasoning
cot = dspy.ChainOfThought(TranslateSignature)

# Retrieve and generate (RAG)
class RAGModule(dspy.Module):
    def __init__(self, retriever):
        super().__init__()
        self.retrieve = retriever
        self.generate = dspy.ChainOfThought("question, context -> answer")
    
    def forward(self, question):
        context = self.retrieve(question, k=3)
        return self.generate(question=question, context=context)

# Multi-step reasoning
class MultiStepReasoner(dspy.Module):
    def __init__(self):
        super().__init__()
        self.analyze = dspy.ChainOfThought("problem -> analysis, sub_problems")
        self.solve = dspy.ChainOfThought("sub_problem -> solution")
        self.synthesize = dspy.ChainOfThought("solutions -> final_answer")
    
    def forward(self, problem):
        analysis = self.analyze(problem=problem)
        solutions = [self.solve(sub_problem=sp) for sp in analysis.sub_problems]
        return self.synthesize(solutions=solutions)
```

## Optimization (The Magic of DSPy)

DSPy's real power comes from 14 different automatic optimization algorithms:

### Few-Shot Learning Optimizers
```python
# Create training examples
trainset = [
    dspy.Example(
        text="Hello world",
        source_language="English", 
        target_language="Spanish",
        translation="Hola mundo"
    ).with_inputs("text", "source_language", "target_language"),
    # Add more examples...
]

# Bootstrap Few-Shot Learning
bootstrap = dspy.BootstrapFewShot(max_bootstrapped_demos=8)
optimized = bootstrap.compile(student=Translator(), trainset=trainset)

# Bootstrap with Random Search (explores multiple program candidates)
bootstrap_rs = dspy.BootstrapRS(max_bootstrapped_demos=4, max_labeled_demos=16)
optimized = bootstrap_rs.compile(student=module, trainset=trainset)

# K-Nearest Neighbor Few-Shot (dynamic demo selection)
knn = dspy.KNNFewShot(k=3, trainset=trainset)
optimized = knn.compile(student=module)
```

### Instruction Optimization
```python
# COPRO: Coordinate Prompt Optimization
copro = dspy.COPRO(metric=accuracy, breadth=10, depth=3)
optimized = copro.compile(student=module, trainset=trainset)

# Auto-generates better instructions based on performance feedback
```

### State-of-the-Art Multi-Modal Optimization
```python
# MIPROv2: Multi-prompt Instruction Proposal Optimizer
mipro = dspy.MIPROv2(
    metric=accuracy,
    num_candidates=25,       # Number of instruction candidates
    init_temperature=1.4,    # Creativity in instruction generation
    verbose=True
)
optimized = mipro.compile(
    student=module, 
    trainset=trainset,
    num_trials=100,         # Bayesian optimization trials
    max_errors=5,
    requires_permission_to_run=False
)

# SIMBA: Stochastic Introspective Mini-Batch Ascent
simba = dspy.SIMBA(
    metric=accuracy,
    num_candidates=7,
    max_num_traces=3       # Multiple execution traces per example
)
optimized = simba.compile(student=module, trainset=trainset)
```

### Fine-Tuning and Weight Optimization
```python
# Fine-tune model weights
finetune = dspy.BootstrapFinetune(
    metric=accuracy,
    target='openai/gpt-3.5-turbo',
    epochs=1,
    multitask=False
)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/brishin) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
