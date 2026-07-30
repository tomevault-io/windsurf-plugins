---
trigger: always_on
description: Security patterns for the Unstructured library used in RAG document ingestion pipelines.
---

# Unstructured Document Processing Security Rules

Security patterns for the Unstructured library used in RAG document ingestion pipelines.

---

## Quick Reference

| Rule | Level | Risk | Primary Defense |
|------|-------|------|-----------------|
| Partition Function Security | `strict` | Resource exhaustion, DoS | Strategy limits, resource controls |
| PDF Processing Security | `strict` | Memory exhaustion, OCR abuse | Mode selection, OCR controls |
| Table Extraction Security | `warning` | Memory exhaustion, malformed data | Size limits, structure validation |
| HTML Processing Security | `strict` | XSS, script injection | Script removal, sanitization |
| Image Extraction Security | `warning` | Metadata leakage, decompression bombs | EXIF stripping, dimension limits |
| Element Metadata Security | `warning` | PII exposure, data leakage | PII filtering, field validation |
| Chunking Strategy Security | `warning` | Context manipulation, resource abuse | Size limits, overlap control |
| API Service Security | `strict` | Credential exposure, rate limit abuse | Authentication, rate limiting |
| Output Validation | `warning` | Injection payloads, data overflow | Element limits, content filtering |

---

## Rule: Partition Function Security

**Level**: `strict`

**When**: Using `partition()`, `partition_pdf()`, `partition_html()`, or other partition functions

**Do**:
```python
from unstructured.partition.auto import partition
from unstructured.partition.pdf import partition_pdf
import os
import resource
from typing import List, Optional
from dataclasses import dataclass

@dataclass
class PartitionSecurityConfig:
    """Security configuration for partition operations."""
    max_file_size_mb: int = 100
    max_pages: int = 500
    max_elements: int = 10000
    timeout_seconds: int = 300
    max_memory_mb: int = 2048
    allowed_strategies: tuple = ("fast", "hi_res", "ocr_only")
    default_strategy: str = "fast"

class SecurePartitioner:
    """Secure wrapper for Unstructured partition functions."""

    def __init__(self, config: Optional[PartitionSecurityConfig] = None):
        self.config = config or PartitionSecurityConfig()

    def partition_document(
        self,
        filename: str,
        strategy: Optional[str] = None,
        **kwargs
    ) -> List:
        """Partition document with security controls."""

        # Validate file size
        file_size_mb = os.path.getsize(filename) / (1024 * 1024)
        if file_size_mb > self.config.max_file_size_mb:
            raise ValueError(
                f"File size {file_size_mb:.1f}MB exceeds limit of "
                f"{self.config.max_file_size_mb}MB"
            )

        # Validate strategy
        if strategy is None:
            strategy = self.config.default_strategy

        if strategy not in self.config.allowed_strategies:
            raise ValueError(
                f"Strategy '{strategy}' not allowed. "
                f"Allowed: {self.config.allowed_strategies}"
            )

        # Set resource limits
        soft, hard = resource.getrlimit(resource.RLIMIT_AS)
        resource.setrlimit(
            resource.RLIMIT_AS,
            (self.config.max_memory_mb * 1024 * 1024, hard)
        )

        try:
            # Partition with controlled parameters
            elements = partition(
                filename=filename,
                strategy=strategy,
                max_partition_length=50000,  # Limit element size
                include_page_breaks=True,
                **kwargs
            )

            # Validate output
            if len(elements) > self.config.max_elements:
                raise ValueError(
                    f"Document produced {len(elements)} elements, "
                    f"exceeds limit of {self.config.max_elements}"
                )

            return list(elements)

        finally:
            # Restore resource limits
            resource.setrlimit(resource.RLIMIT_AS, (soft, hard))

    def partition_with_timeout(
        self,
        filename: str,
        **kwargs
    ) -> List:
        """Partition with timeout protection."""
        import signal

        def timeout_handler(signum, frame):
            raise TimeoutError(
                f"Partition exceeded {self.config.timeout_seconds}s timeout"
            )

        old_handler = signal.signal(signal.SIGALRM, timeout_handler)
        signal.alarm(self.config.timeout_seconds)

        try:
            return self.partition_document(filename, **kwargs)
        finally:
            signal.alarm(0)
            signal.signal(signal.SIGALRM, old_handler)
```

**Don't**:
```python
# VULNERABLE: No resource controls
from unstructured.partition.auto import partition

def process_document(filename):
    # No file size check - can process 10GB files
    # No strategy validation - hi_res on all docs (expensive)
    # No element limit - can return millions of elements
    # No timeout - can run forever

    elements = partition(filename)  # Uncontrolled execution
    return elements

# VULNERABLE: Using hi_res for all documents
elements = partition(filename, strategy="hi_res")  # Resource intensive
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TikiTribe/claude-secure-coding-rules](https://github.com/TikiTribe/claude-secure-coding-rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
