---
trigger: always_on
description: Detect and prevent security misconfigurations in JavaScript applications as defined in OWASP Top 10:2021-A05
---

# JavaScript Security Misconfiguration (OWASP A05:2021)

<rule>
name: javascript_security_misconfiguration
description: Detect and prevent security misconfigurations in JavaScript applications as defined in OWASP Top 10:2021-A05

actions:
  - type: enforce
    conditions:
      # Pattern 1: Missing or Insecure HTTP Security Headers
      - pattern: "app\\.use\\([^)]*?\\)\\s*(?!.*(?:helmet|frameguard|hsts|noSniff|xssFilter|contentSecurityPolicy))"
        location: "(?:app|server|index)\\.(?:js|ts)$"
        message: "Missing HTTP security headers. Consider using Helmet.js to set secure HTTP headers."
        
      # Pattern 2: Insecure CORS Configuration
      - pattern: "app\\.use\\(cors\\(\\{[^}]*?origin\\s*:\\s*['\"]\\*['\"]\\s*\\}\\)\\)"
        message: "Insecure CORS configuration. Avoid using wildcard (*) for CORS origin in production environments."
        
      # Pattern 3: Exposed Environment Variables in Client-Side Code
      - pattern: "process\\.env\\.[A-Z_]+"
        location: "(?:src|components|pages)"
        message: "Exposing environment variables in client-side code. Only use environment variables with NEXT_PUBLIC_, REACT_APP_, or VITE_ prefixes for client-side code."
        
      # Pattern 4: Insecure Cookie Settings
      - pattern: "(?:cookie|cookies|session)\\([^)]*?\\{[^}]*?(?:secure\\s*:\\s*false|httpOnly\\s*:\\s*false|sameSite\\s*:\\s*['\"]none['\"])"
        message: "Insecure cookie configuration. Set secure:true, httpOnly:true, and appropriate sameSite value for cookies."
        
      # Pattern 5: Missing Content Security Policy
      - pattern: "app\\.use\\([^)]*?helmet\\([^)]*?\\{[^}]*?contentSecurityPolicy\\s*:\\s*false"
        message: "Content Security Policy (CSP) is disabled. Enable and configure CSP to prevent XSS attacks."
        
      # Pattern 6: Debug Information Exposure
      - pattern: "app\\.use\\([^)]*?morgan\\(['\"]dev['\"]\\)|console\\.(?:log|debug|info|warn|error)\\("
        location: "(?:app|server|index)\\.(?:js|ts)$"
        message: "Debug information might be exposed in production. Ensure logging is properly configured based on the environment."
        
      # Pattern 7: Insecure Server Configuration
      - pattern: "app\\.disable\\(['\"]x-powered-by['\"]\\)"
        negative_pattern: true
        location: "(?:app|server|index)\\.(?:js|ts)$"
        message: "X-Powered-By header is not disabled. Use app.disable('x-powered-by') to hide technology information."
        
      # Pattern 8: Directory Listing Enabled
      - pattern: "express\\.static\\([^)]*?\\{[^}]*?index\\s*:\\s*false"
        message: "Directory listing might be enabled. Set index:true or provide an index file to prevent directory listing."
        
      # Pattern 9: Missing Rate Limiting
      - pattern: "app\\.(?:get|post|put|delete|patch)\\([^)]*?['\"](?:/api|/login|/register|/auth)['\"]"
        negative_pattern: "(?:rateLimit|rateLimiter|limiter|throttle)"
        message: "Missing rate limiting for sensitive endpoints. Implement rate limiting to prevent brute force attacks."
        
      # Pattern 10: Insecure WebSocket Configuration
      - pattern: "new\\s+WebSocket\\([^)]*?\\)|io\\.on\\(['\"]connection['\"]"
        negative_pattern: "(?:wss://|https://)"
        message: "Potentially insecure WebSocket connection. Use secure WebSocket (wss://) in production."
        
      # Pattern 11: Hardcoded Configuration Values
      - pattern: "(?:apiKey|secret|password|token|credentials)\\s*=\\s*['\"][^'\"]+['\"]"
        message: "Hardcoded configuration values. Use environment variables or a secure configuration management system."
        
      # Pattern 12: Insecure SSL/TLS Configuration
      - pattern: "https\\.createServer\\([^)]*?\\{[^}]*?rejectUnauthorized\\s*:\\s*false"
        message: "Insecure SSL/TLS configuration. Never set rejectUnauthorized:false in production."
        
      # Pattern 13: Missing Security Middleware
      - pattern: "express\\(\\)|require\\(['\"]express['\"]\\)"
        negative_pattern: "(?:helmet|cors|rateLimit|bodyParser\\.json\\(\\{\\s*limit|express\\.json\\(\\{\\s*limit)"
        location: "(?:app|server|index)\\.(?:js|ts)$"
        message: "Missing essential security middleware. Consider using helmet, cors, rate limiting, and request size limiting."
        
      # Pattern 14: Insecure Error Handling
      - pattern: "app\\.use\\([^)]*?function\\s*\\([^)]*?err[^)]*?\\)\\s*\\{[^}]*?res\\.status[^}]*?err(?:\\.message|\\.stack)"
        message: "Insecure error handling. Avoid exposing error details like stack traces to clients in production."
        
      # Pattern 15: Outdated Dependencies Warning
      - pattern: "(?:\"dependencies\"|\"devDependencies\")\\s*:\\s*\\{[^}]*?['\"](?:express|react|vue|angular|next|nuxt|axios)['\"]\\s*:\\s*['\"]\\^?\\d+\\.\\d+\\.\\d+['\"]"
        location: "package\\.json$"
        message: "Check for outdated dependencies. Regularly update dependencies to avoid known vulnerabilities."

  - type: suggest
    message: |
      **JavaScript Security Configuration Best Practices:**
      
      1. **HTTP Security Headers:**
         - Use Helmet.js to set secure HTTP headers
         - Configure Content Security Policy (CSP)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ivangrynenko/cursorrules](https://github.com/ivangrynenko/cursorrules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
