---
trigger: always_on
description: Detect and prevent Server-Side Request Forgery (SSRF) vulnerabilities in Python applications as defined in OWASP Top 10:2021-A10
---

 # Python Server-Side Request Forgery (SSRF) Standards (OWASP A10:2021)

This rule enforces security best practices to prevent Server-Side Request Forgery (SSRF) vulnerabilities in Python applications, as defined in OWASP Top 10:2021-A10.

<rule>
name: python_ssrf
description: Detect and prevent Server-Side Request Forgery (SSRF) vulnerabilities in Python applications as defined in OWASP Top 10:2021-A10
filters:
  - type: file_extension
    pattern: "\\.py$"
  - type: file_path
    pattern: ".*"

actions:
  - type: enforce
    conditions:
      # Pattern 1: Detect direct use of requests library with user input
      - pattern: "requests\\.(get|post|put|delete|head|options|patch)\\([^)]*?\\b(request\\.\\w+|params\\[\\'[^\\']+\\'\\]|data\\[\\'[^\\']+\\'\\]|json\\[\\'[^\\']+\\'\\]|args\\.get|form\\.get)"
        message: "Potential SSRF vulnerability detected. User-controlled input is being used directly in HTTP requests. Implement URL validation and allowlisting."
        
      # Pattern 2: Detect urllib usage with user input
      - pattern: "urllib\\.(request|parse)\\.\\w+\\([^)]*?\\b(request\\.\\w+|params\\[\\'[^\\']+\\'\\]|data\\[\\'[^\\']+\\'\\]|json\\[\\'[^\\']+\\'\\]|args\\.get|form\\.get)"
        message: "Potential SSRF vulnerability detected. User-controlled input is being used directly in urllib functions. Implement URL validation and allowlisting."
        
      # Pattern 3: Detect http.client usage with user input
      - pattern: "http\\.client\\.\\w+\\([^)]*?\\b(request\\.\\w+|params\\[\\'[^\\']+\\'\\]|data\\[\\'[^\\']+\\'\\]|json\\[\\'[^\\']+\\'\\]|args\\.get|form\\.get)"
        message: "Potential SSRF vulnerability detected. User-controlled input is being used directly in http.client functions. Implement URL validation and allowlisting."
        
      # Pattern 4: Detect aiohttp usage with user input
      - pattern: "aiohttp\\.\\w+\\([^)]*?\\b(request\\.\\w+|params\\[\\'[^\\']+\\'\\]|data\\[\\'[^\\']+\\'\\]|json\\[\\'[^\\']+\\'\\]|args\\.get|form\\.get)"
        message: "Potential SSRF vulnerability detected. User-controlled input is being used directly in aiohttp functions. Implement URL validation and allowlisting."
        
      # Pattern 5: Detect httpx usage with user input
      - pattern: "httpx\\.\\w+\\([^)]*?\\b(request\\.\\w+|params\\[\\'[^\\']+\\'\\]|data\\[\\'[^\\']+\\'\\]|json\\[\\'[^\\']+\\'\\]|args\\.get|form\\.get)"
        message: "Potential SSRF vulnerability detected. User-controlled input is being used directly in httpx functions. Implement URL validation and allowlisting."
        
      # Pattern 6: Detect pycurl usage with user input
      - pattern: "pycurl\\.\\w+\\([^)]*?\\b(request\\.\\w+|params\\[\\'[^\\']+\\'\\]|data\\[\\'[^\\']+\\'\\]|json\\[\\'[^\\']+\\'\\]|args\\.get|form\\.get)"
        message: "Potential SSRF vulnerability detected. User-controlled input is being used directly in pycurl functions. Implement URL validation and allowlisting."
        
      # Pattern 7: Detect subprocess calls with user input that might lead to SSRF
      - pattern: "subprocess\\.(Popen|call|run|check_output|check_call)\\([^)]*?\\b(request\\.\\w+|params\\[\\'[^\\']+\\'\\]|data\\[\\'[^\\']+\\'\\]|json\\[\\'[^\\']+\\'\\]|args\\.get|form\\.get)"
        message: "Potential SSRF vulnerability detected. User-controlled input is being used in subprocess calls, which might lead to SSRF. Validate and sanitize input."
        
      # Pattern 8: Detect os.system calls with user input that might lead to SSRF
      - pattern: "os\\.(system|popen|spawn)\\([^)]*?\\b(request\\.\\w+|params\\[\\'[^\\']+\\'\\]|data\\[\\'[^\\']+\\'\\]|json\\[\\'[^\\']+\\'\\]|args\\.get|form\\.get)"
        message: "Potential SSRF vulnerability detected. User-controlled input is being used in OS commands, which might lead to SSRF. Validate and sanitize input."
        
      # Pattern 9: Detect URL construction with user input
      - pattern: "(f|r)[\"\']https?://[^\"\']*?\\{[^\\}]*?\\b(request\\.\\w+|params\\[\\'[^\\']+\\'\\]|data\\[\\'[^\\']+\\'\\]|json\\[\\'[^\\']+\\'\\]|args\\.get|form\\.get)"
        message: "Potential SSRF vulnerability detected. User-controlled input is being used in URL construction. Implement URL validation and allowlisting."
        
      # Pattern 10: Detect URL joining with user input
      - pattern: "urljoin\\([^,]+,[^)]*?\\b(request\\.\\w+|params\\[\\'[^\\']+\\'\\]|data\\[\\'[^\\']+\\'\\]|json\\[\\'[^\\']+\\'\\]|args\\.get|form\\.get)"
        message: "Potential SSRF vulnerability detected. User-controlled input is being used in URL joining. Implement URL validation and allowlisting."
        
      # Pattern 11: Detect file opening with user input (potential local SSRF)
      - pattern: "open\\([^,]*?\\b(request\\.\\w+|params\\[\\'[^\\']+\\'\\]|data\\[\\'[^\\']+\\'\\]|json\\[\\'[^\\']+\\'\\]|args\\.get|form\\.get)"
        message: "Potential local SSRF vulnerability detected. User-controlled input is being used in file operations. Validate file paths and use path sanitization."
        
      # Pattern 12: Detect XML/YAML parsing with user input (potential XXE leading to SSRF)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ivangrynenko/cursorrules](https://github.com/ivangrynenko/cursorrules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
