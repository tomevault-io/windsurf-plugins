---
trigger: always_on
description: Detect and prevent Server-Side Request Forgery (SSRF) vulnerabilities in JavaScript applications as defined in OWASP Top 10:2021-A10
---

# JavaScript Server-Side Request Forgery (OWASP A10:2021)

<rule>
name: javascript_server_side_request_forgery
description: Detect and prevent Server-Side Request Forgery (SSRF) vulnerabilities in JavaScript applications as defined in OWASP Top 10:2021-A10

actions:
  - type: enforce
    conditions:
      # Pattern 1: URL from User Input
      - pattern: "(fetch|axios\\.get|axios\\.post|axios\\.put|axios\\.delete|axios\\.patch|http\\.get|http\\.request|https\\.get|https\\.request|\\$\\.ajax|XMLHttpRequest|got|request|superagent|needle)\\s*\\([^)]*(?:\\$_GET|\\$_POST|\\$_REQUEST|req\\.(?:body|query|params)|request\\.(?:body|query|params)|event\\.(?:body|queryStringParameters|pathParameters)|params|userInput|data\\["
        message: "Potential SSRF vulnerability: URL constructed from user input. Implement URL validation, allowlisting, or use a URL parser library to validate and sanitize user-provided URLs."
        
      # Pattern 2: Dynamic URL in HTTP Request
      - pattern: "(fetch|axios|http\\.get|http\\.request|https\\.get|https\\.request|\\$\\.ajax|XMLHttpRequest|got|request|superagent|needle)\\s*\\(\\s*['\"`]https?:\\/\\/[^'\"`]*['\"`]\\s*\\+\\s*"
        message: "Potential SSRF vulnerability: Dynamic URL in HTTP request. Use URL parsing and validation before making the request."
        
      # Pattern 3: URL Redirection Without Validation
      - pattern: "(res\\.redirect|res\\.location|window\\.location|location\\.href|location\\.replace|location\\.assign|location\\.port|history\\.pushState|history\\.replaceState)\\s*\\([^)]*(?:req\\.(?:query|body|params)|request\\.(?:query|body|params)|userInput)"
        message: "URL redirection without proper validation may lead to SSRF. Implement strict validation for URLs before redirecting."
        
      # Pattern 4: Direct IP Address Usage
      - pattern: "(fetch|axios\\.get|axios\\.post|axios\\.put|axios\\.delete|axios\\.patch|http\\.get|http\\.request|https\\.get|https\\.request)\\s*\\(\\s*['\"`]https?:\\/\\/\\d{1,3}\\.\\d{1,3}\\.\\d{1,3}\\.\\d{1,3}"
        message: "Direct use of IP addresses in requests may bypass hostname-based restrictions. Consider using allowlisted hostnames instead."
        
      # Pattern 5: Local Network Access
      - pattern: "(fetch|axios\\.get|axios\\.post|axios\\.put|axios\\.delete|axios\\.patch|http\\.get|http\\.request|https\\.get|https\\.request)\\s*\\(\\s*['\"`]https?:\\/\\/(?:localhost|127\\.0\\.0\\.1|0\\.0\\.0\\.0|192\\.168\\.|10\\.|172\\.(?:1[6-9]|2[0-9]|3[0-1])\\.|::1)"
        message: "Request to internal network address detected. Restrict access to internal resources to prevent SSRF attacks."
        
      # Pattern 6: File Protocol Usage
      - pattern: "(fetch|axios\\.get|axios\\.post|axios\\.put|axios\\.delete|axios\\.patch|http\\.get|http\\.request|https\\.get|https\\.request)\\s*\\(\\s*['\"`]file:\\/\\/"
        message: "Use of file:// protocol may lead to local file access. Block or restrict file:// protocol usage."
        
      # Pattern 7: Missing URL Validation
      - pattern: "(fetch|axios\\.get|axios\\.post|axios\\.put|axios\\.delete|axios\\.patch|http\\.get|http\\.request|https\\.get|https\\.request)\\s*\\([^)]*\\burl\\b[^)]*\\)"
        negative_pattern: "(validat|sanitiz|check|parse).*\\burl\\b|allowlist|whitelist|URL\\.(parse|canParse)|new URL\\(|isValidURL"
        message: "HTTP request without URL validation. Implement URL validation before making external requests."
        
      # Pattern 8: HTTP Request in User-Defined Function
      - pattern: "function\\s+[a-zA-Z0-9_]*(?:request|fetch|get|http|curl)\\s*\\([^)]*\\)\\s*\\{[^}]*(?:fetch|axios|http\\.get|http\\.request|https\\.get|https\\.request)"
        negative_pattern: "(validat|sanitiz|check|parse).*\\burl\\b|allowlist|whitelist|new URL\\(|isValidURL"
        message: "User-defined HTTP request function without URL validation. Implement proper URL validation and sanitization."
        
      # Pattern 9: Proxy Functionality
      - pattern: "(?:proxy|forward|relay).*(?:req\\.(?:url|path)|request\\.(?:url|path))"
        negative_pattern: "(validat|sanitiz|check|parse).*\\burl\\b|allowlist|whitelist"
        message: "Proxy or request forwarding functionality detected. Implement strict URL validation and allowlisting."
        
      # Pattern 10: Alternative HTTP Methods
      - pattern: "(fetch|axios)\\s*\\([^)]*method\\s*:\\s*['\"`](?:GET|POST|PUT|DELETE|PATCH|OPTIONS|HEAD)['\"`]"
        negative_pattern: "(validat|sanitiz|check|parse).*\\burl\\b|allowlist|whitelist|new URL\\(|isValidURL"
        message: "HTTP request with explicit method without URL validation. Implement URL validation for all HTTP methods."
        
      # Pattern 11: URL Building from Parts
      - pattern: "new URL\\s*\\((?:[^,)]+,\\s*){1,}(?:req\\.(?:body|query|params)|request\\.(?:body|query|params)|userinput)"
        message: "Building URL with user input. Validate and sanitize all URL components and use an allowlist for base URLs."
        
      # Pattern 12: Protocol-Relative URLs
      - pattern: "(fetch|axios)\\s*\\(['\"`]\\/\\/[^'\"`]+['\"`]"
        message: "Protocol-relative URL usage may lead to SSRF. Always specify the protocol and validate URLs."

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ivangrynenko/cursorrules](https://github.com/ivangrynenko/cursorrules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
