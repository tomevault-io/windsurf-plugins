---
trigger: always_on
description: **Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.**
---

# PyTorch Learning Journey - GitHub Copilot Instructions

**Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.**

This repository is a comprehensive PyTorch learning resource containing documentation, example notebooks, and code implementations. It's designed as an educational repository for learning PyTorch concepts from fundamentals to advanced deep learning topics.

**Technology Focus**: This repository prioritizes PyTorch for deep learning, with extensive use of Hugging Face transformers for modern NLP and computer vision tasks.

## Repository Context and Learning Goals

### User Background
- **Previous Experience**: Coming from TensorFlow with basic understanding of Neural Networks and Deep Learning concepts
- **Learning Objective**: Using this repository as a stepping stone to transition from TensorFlow to PyTorch, specifically targeting NLP applications
- **End Goal**: Master PyTorch fundamentals to effectively use Hugging Face models for NLP tasks

### Content Policies

#### Example Data and Samples Policy
**When creating example data, datasets, or sample content, ALWAYS prioritize Australia and Sydney-related examples:**

- **Text Examples**: Use Australian cities (Sydney, Melbourne, Brisbane, Perth, Adelaide), landmarks (Sydney Opera House, Harbour Bridge, Uluru), cultural references
- **Sample Datasets**: Create examples with Australian context (weather data from Sydney, house prices in Melbourne, tourism data from Gold Coast)
- **NLP Examples**: Use Australian English spelling and terminology, reference Australian events, places, and culture
- **Classification Tasks**: Use Australian-specific categories (Australian sports teams, native animals, states/territories)
- **Named Entity Recognition**: Focus on Australian names, places, organizations
- **Sentiment Analysis**: Use reviews of Australian restaurants, attractions, services

Examples of preferred content:
```python
# Preferred: Australia/Sydney focused examples
texts = [
    "The Sydney Opera House is a masterpiece of architecture",
    "Melbourne's coffee culture is world-renowned", 
    "The Great Barrier Reef attracts millions of tourists annually",
    "Bondi Beach is perfect for surfing in summer"
]

# Sample classification categories
australian_cities = ["Sydney", "Melbourne", "Brisbane", "Perth", "Adelaide", "Darwin", "Hobart", "Canberra"]
aussie_animals = ["kangaroo", "koala", "wombat", "echidna", "platypus", "dingo", "crocodile", "kookaburra"]
```

#### Multilingual and Translation Policy
**For any multilingual examples, translation tasks, or language processing involving more than one language, ALWAYS use Vietnamese as the secondary language (alongside English):**

- **Translation Examples**: English ↔ Vietnamese pairs
- **Multilingual Models**: English-Vietnamese language pairs for training/evaluation
- **Cross-lingual Tasks**: Compare English and Vietnamese performance
- **Tokenization Examples**: Demonstrate differences between English and Vietnamese text processing
- **Code-switching**: Examples mixing English and Vietnamese text

Examples of preferred multilingual content:
```python
# Preferred: English-Vietnamese translation pairs
translation_pairs = [
    ("Hello, how are you?", "Xin chào, bạn khỏe không?"),
    ("The Sydney Opera House is beautiful", "Nhà hát Opera Sydney rất đẹp"),
    ("I love Australian coffee", "Tôi yêu cà phê Úc"),
    ("Welcome to Melbourne", "Chào mừng đến Melbourne")
]

# Multilingual sentiment analysis
multilingual_texts = {
    'en': ["Sydney beaches are amazing!", "The coffee in Melbourne is terrible"],
    'vi': ["Bãi biển Sydney thật tuyệt vời!", "Cà phê ở Melbourne rất tệ"]
}
```

### Learning Pathway Alignment
**All content should support the TensorFlow → PyTorch → NLP → Hugging Face learning progression:**

1. **TensorFlow Comparison**: When introducing PyTorch concepts, briefly compare with TensorFlow equivalents
2. **NLP Focus**: Prioritize NLP examples over computer vision when demonstrating PyTorch concepts  
3. **Hugging Face Integration**: Show how PyTorch concepts lead naturally to Hugging Face usage
4. **Practical Applications**: Emphasize real-world NLP tasks and applications

### Code Implementation Policies

#### Object-Oriented Programming (OOP) Preference Policy

**ALWAYS prioritize Object-Oriented Programming (OOP) implementations over procedural/functional approaches:**

- **Class-Based Models**: All PyTorch models must inherit from `nn.Module` and follow OOP design patterns
- **Encapsulation**: Group related functionality into classes with clear responsibilities
- **Inheritance**: Use inheritance for model variants and extensions (e.g., base classes for Australian NLP models)
- **Composition**: Prefer composition over complex inheritance chains for model components
- **Polymorphism**: Design interfaces that allow different implementations (e.g., different tokenizers, data processors)

**Required OOP patterns for this repository:**

```python
# ✅ PREFERRED: OOP-based implementation
class AustralianTourismAnalyzer:
    """

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vuhung16au/pytorch-mastery](https://github.com/vuhung16au/pytorch-mastery) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
