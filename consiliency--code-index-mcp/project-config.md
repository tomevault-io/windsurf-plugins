---
trigger: always_on
description: This file defines security policies, best practices, and implementation guidelines for the Code-Index-MCP project to ensure safe code indexing and analysis.
---

# Security Rules for Code-Index-MCP

## Overview
This file defines security policies, best practices, and implementation guidelines for the Code-Index-MCP project to ensure safe code indexing and analysis.

## Input Validation

### File Path Validation
```python
import os
from pathlib import Path

def validate_file_path(file_path: str, base_dir: str) -> bool:
    """Ensure file paths are within allowed directories."""
    try:
        # Resolve to absolute path
        abs_path = Path(file_path).resolve()
        base_path = Path(base_dir).resolve()
        
        # Check if path is within base directory
        return abs_path.is_relative_to(base_path)
    except Exception:
        return False

# Usage in API endpoints
@app.post("/index")
async def index_file(file_path: str):
    if not validate_file_path(file_path, os.getcwd()):
        raise HTTPException(403, "Access denied: Invalid file path")
```

### Query Sanitization
```python
def sanitize_search_query(query: str) -> str:
    """Remove potentially dangerous characters from search queries."""
    # Remove SQL injection attempts
    dangerous_patterns = [';', '--', '/*', '*/', 'xp_', 'sp_']
    for pattern in dangerous_patterns:
        query = query.replace(pattern, '')
    
    # Limit query length
    return query[:1000]
```

## Authentication & Authorization

### API Key Management
```python
import secrets
import hashlib

class APIKeyManager:
    def generate_api_key(self) -> str:
        """Generate secure API key."""
        return secrets.token_urlsafe(32)
    
    def hash_api_key(self, api_key: str) -> str:
        """Hash API key for storage."""
        return hashlib.sha256(api_key.encode()).hexdigest()
    
    def verify_api_key(self, provided_key: str, stored_hash: str) -> bool:
        """Verify API key against stored hash."""
        return self.hash_api_key(provided_key) == stored_hash
```

### Request Authentication
```python
from fastapi import Security, HTTPException
from fastapi.security import APIKeyHeader

api_key_header = APIKeyHeader(name="X-API-Key")

async def verify_api_key(api_key: str = Security(api_key_header)):
    """Verify API key middleware."""
    if not api_key_manager.verify_api_key(api_key):
        raise HTTPException(401, "Invalid API key")
    return api_key
```

## Secret Detection

### Pattern Detection
```python
import re

SECRET_PATTERNS = {
    'aws_access_key': r'AKIA[0-9A-Z]{16}',
    'aws_secret_key': r'[0-9a-zA-Z/+=]{40}',
    'github_token': r'ghp_[0-9a-zA-Z]{36}',
    'api_key': r'api[_-]?key[_-]?[:=]\s*["\']?([0-9a-zA-Z\-_]+)["\']?',
    'private_key': r'-----BEGIN (RSA|DSA|EC|OPENSSH) PRIVATE KEY-----'
}

def detect_secrets(content: str) -> List[Dict]:
    """Detect potential secrets in code."""
    findings = []
    for secret_type, pattern in SECRET_PATTERNS.items():
        matches = re.finditer(pattern, content, re.IGNORECASE)
        for match in matches:
            findings.append({
                'type': secret_type,
                'line': content[:match.start()].count('\n') + 1,
                'matched': match.group(0)[:20] + '...'  # Truncate for safety
            })
    return findings
```

### Secret Redaction
```python
def redact_secrets(content: str) -> str:
    """Redact detected secrets from content."""
    for secret_type, pattern in SECRET_PATTERNS.items():
        content = re.sub(pattern, f'[REDACTED_{secret_type.upper()}]', content, flags=re.IGNORECASE)
    return content
```

## Plugin Security

### Plugin Isolation
```python
import subprocess
import resource

def run_plugin_safely(plugin_path: str, input_data: str) -> str:
    """Run plugin in isolated environment."""
    # Set resource limits
    def set_limits():
        # Limit CPU time (seconds)
        resource.setrlimit(resource.RLIMIT_CPU, (5, 5))
        # Limit memory (bytes)
        resource.setrlimit(resource.RLIMIT_AS, (512 * 1024 * 1024, 512 * 1024 * 1024))
    
    # Run plugin as subprocess with limits
    result = subprocess.run(
        ['python', plugin_path],
        input=input_data,
        capture_output=True,
        text=True,
        timeout=10,
        preexec_fn=set_limits
    )
    
    return result.stdout
```

### Plugin Validation
```python
def validate_plugin(plugin_class):
    """Validate plugin implements required interface."""
    required_methods = ['index', 'getDefinition', 'getReferences']
    
    for method in required_methods:
        if not hasattr(plugin_class, method):
            raise SecurityError(f"Plugin missing required method: {method}")
    
    # Check for dangerous operations
    source = inspect.getsource(plugin_class)
    dangerous_ops = ['eval', 'exec', '__import__', 'compile']
    
    for op in dangerous_ops:
        if op in source:
            raise SecurityError(f"Plugin contains dangerous operation: {op}")
```

## Network Security

### HTTPS Enforcement
```python
from fastapi.middleware.httpsredirect import HTTPSRedirectMiddleware

# Force HTTPS in production
if os.getenv("ENVIRONMENT") == "production":
    app.add_middleware(HTTPSRedirectMiddleware)
```

### CORS Configuration
```python
from fastapi.middleware.cors import CORSMiddleware


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Consiliency/Code-Index-MCP](https://github.com/Consiliency/Code-Index-MCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-16 -->
