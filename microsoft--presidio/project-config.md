---
trigger: always_on
description: Presidio is a Python-based data protection and de-identification SDK with multiple components for detecting and anonymizing PII (Personally Identifiable Information) in text and images.
---

# Presidio Development Instructions

Presidio is a Python-based data protection and de-identification SDK with multiple components for detecting and anonymizing PII (Personally Identifiable Information) in text and images.

## Core Philosophy

**Data Privacy is Paramount** - This is a PII detection and anonymization system used in sensitive contexts. Security and correctness are non-negotiable.

**Key Principles:**
- **Accuracy First**: False negatives (missed PII) and false positives (incorrect detections) both damage trust
- **Security by Default**: Never log PII values, use non-reversible anonymization, validate all inputs
- **Cross-Component Awareness**: Presidio is a multi-component system - changes ripple across boundaries
- **Stateless Design**: Presidio is designed for scalability - avoid adding unnecessary state
- **Documentation Integrity**: Code and docs must stay synchronized - outdated docs are dangerous

---

## Part 1: Implementation Guidelines

Use these guidelines when **generating or writing code** for Presidio.

### Presidio Architecture Patterns



**Data Flow (Unidirectional):**
```
Analyzer (detect PII) → Anonymizer (transform PII) → Output
    ↓
nlp_engine → recognizers → context
```

**Design Patterns:**
- **Registry Pattern**: `RecognizerRegistry` for dynamic recognizer management
- **Provider Pattern**: `NlpEngineProvider`, `RecognizerRegistryProvider` for configuration

### Implementing New Recognizers

**1. Choose the Right Base Class:**
```python
from presidio_analyzer import PatternRecognizer, LocalRecognizer, RemoteRecognizer

# For regex-based detection
class MyPatternRecognizer(PatternRecognizer):
    pass

# For custom logic (NLP, ML)
class MyCustomRecognizer(LocalRecognizer):
    def load(self): ...
    def analyze(self, text, entities, nlp_artifacts): ...

# For calling remote services
class MyRemoteRecognizer(RemoteRecognizer):
    def analyze(self, text, entities, nlp_artifacts): ...
```

**2. Predefined Recognizers Location Matters:**
- Country-specific: `presidio-analyzer/presidio_analyzer/predefined_recognizers/country_specific/{country}/`
- Generic patterns: `.../predefined_recognizers/generic/`
- NLP/ML-based: `.../predefined_recognizers/nlp_engine_recognizers/` or `.../ner/`
- Third-party: `.../predefined_recognizers/third_party/`

**3. Pattern Design Best Practices:**
```python
# ❌ BAD: Too broad - matches month names as persons
pattern = r"\b[A-Z][a-z]+\b"

# ✅ GOOD: Specific pattern with context
PATTERNS = [
    Pattern(
        "SSN", 
        r"\b\d{3}-\d{2}-\d{4}\b",
        0.3  # Low base score, context will boost
    )
]

CONTEXT = ["ssn", "social security", "tax id"]
```

**4. Document Pattern Sources:**
```python
"""
Recognizes US Social Security Numbers.

Pattern based on SSA Publication No. 05-10633:
https://www.ssa.gov/history/ssn/geocard.html

Validation uses SSN format rules: AAA-GG-SSSS
- AAA: Area number (001-899, excluding 666)
- GG: Group number (01-99)
- SSSS: Serial number (0001-9999)
"""
```

**5. Required Configuration Updates:**
```python
# Update all of these:
# 1. presidio_analyzer/predefined_recognizers/__init__.py
from .country_specific.us.my_recognizer import MyRecognizer
__all__ = [..., "MyRecognizer"]

# 2. presidio_analyzer/predefined_recognizers/country_specific/us/__init__.py  
from .my_recognizer import MyRecognizer
__all__ = [..., "MyRecognizer"]

# 3. presidio_analyzer/conf/default_recognizers.yaml
recognizers:
  - name: MyRecognizer
    supported_languages: ["en"]
    type: predefined
    enabled: false  # Country-specific defaults to false

# 4. docs/supported_entities.md (add row to appropriate table)
# 5. CHANGELOG.md (under "Unreleased" section)
```

**6. Comprehensive Test Coverage:**
```python
@pytest.mark.parametrize("text, expected_len, expected_positions", [
    # True positives - valid formats
    ("SSN: 123-45-6789", 1, ((5, 16),)),
    ("My SSN is 123-45-6789", 1, ((10, 21),)),
    
    # True negatives - invalid formats  
    ("SSN: 000-00-0000", 0, ()),  # Invalid area
    ("SSN: 666-12-3456", 0, ()),  # Excluded area
    
    # Boundary testing - embedded in text
    ("Contact: 123-45-6789 for info", 1, ((9, 20),)),
    
    # False positive prevention
    ("ISBN: 123-45-6789", 0, ()),  # Different context
])
def test_ssn_detection(text, expected_len, expected_positions, recognizer):
    results = recognizer.analyze(text, ["US_SSN"])
    assert len(results) == expected_len
    for result, (start, end) in zip(results, expected_positions):
        assert result.start == start
        assert result.end == end
```

### Implementing New Anonymizers (Operators)

**1. Implement the Operator Interface:**
```python
from presidio_anonymizer.operators import Operator, OperatorType

class MyOperator(Operator):
    """Custom anonymization operator."""
    
    def operate(self, text: str, params: dict = None) -> str:
        """Transform the detected PII."""
        # Ensure non-reversible transformation
        import uuid
        return f"<{params.get('entity_type', 'REDACTED')}_{uuid.uuid4().hex[:8]}>"
    
    def validate(self, params: dict = None) -> None:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/presidio](https://github.com/microsoft/presidio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
