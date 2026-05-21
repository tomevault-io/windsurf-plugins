---
trigger: always_on
description: Detect and prevent insecure design patterns in JavaScript applications as defined in OWASP Top 10:2021-A04
---

# JavaScript Insecure Design (OWASP A04:2021)

<rule>
name: javascript_insecure_design
description: Detect and prevent insecure design patterns in JavaScript applications as defined in OWASP Top 10:2021-A04

actions:
  - type: enforce
    conditions:
      # Pattern 1: Lack of Rate Limiting
      - pattern: "app\\.(?:get|post|put|delete|patch)\\([^)]*?\\)\\s*(?!.*(?:rateLimiter|limiter|throttle|rateLimit))"
        location: "(?:routes|api|controllers)"
        message: "Potential lack of rate limiting in API endpoint. Consider implementing rate limiting to prevent abuse."
        
      # Pattern 2: Insecure Direct Object Reference (IDOR)
      - pattern: "(?:findById|getById|findOne)\\([^)]*?(?:req\\.|request\\.|params\\.|query\\.|body\\.|user\\.|input\\.|form\\.)[^)]*?\\)\\s*(?!.*(?:authorization|permission|access|canAccess|isAuthorized|checkPermission))"
        location: "(?:routes|api|controllers)"
        message: "Potential Insecure Direct Object Reference (IDOR) vulnerability. Implement proper authorization checks before accessing objects by ID."
        
      # Pattern 3: Lack of Input Validation
      - pattern: "(?:req\\.|request\\.|params\\.|query\\.|body\\.|user\\.|input\\.|form\\.)[a-zA-Z0-9_]+\\s*(?!.*(?:validate|sanitize|check|schema|joi|yup|zod|validator|isValid))"
        location: "(?:routes|api|controllers)"
        message: "Potential lack of input validation. Implement proper validation for all user inputs."
        
      # Pattern 4: Hardcoded Business Logic
      - pattern: "if\\s*\\([^)]*?(?:role\\s*===\\s*['\"]admin['\"]|isAdmin\\s*===\\s*true|user\\.role\\s*===\\s*['\"]admin['\"])\\s*\\)"
        message: "Hardcoded business logic for authorization. Consider using a more flexible role-based access control system."
        
      # Pattern 5: Lack of Proper Error Handling
      - pattern: "catch\\s*\\([^)]*?\\)\\s*\\{[^}]*?(?:console\\.(?:log|error))[^}]*?\\}"
        negative_pattern: "(?:res\\.status|next\\(err|next\\(error|errorHandler)"
        message: "Improper error handling. Avoid only logging errors without proper handling or user feedback."
        
      # Pattern 6: Insecure Authentication Design
      - pattern: "(?:password|token|secret|key)\\s*===\\s*(?:req\\.|request\\.|params\\.|query\\.|body\\.|user\\.|input\\.|form\\.)"
        message: "Insecure authentication design. Avoid direct string comparison for passwords or tokens."
        
      # Pattern 7: Lack of Proper Logging
      - pattern: "app\\.(?:get|post|put|delete|patch)\\([^)]*?\\)\\s*(?!.*(?:log|logger|winston|bunyan|morgan|audit))"
        location: "(?:routes|api|controllers)"
        message: "Lack of proper logging in API endpoint. Implement logging for security-relevant events."
        
      # Pattern 8: Insecure Defaults
      - pattern: "new\\s+(?:Session|Cookie|JWT)\\([^)]*?\\{[^}]*?(?:secure\\s*:\\s*false|httpOnly\\s*:\\s*false|sameSite\\s*:\\s*['\"]none['\"])"
        message: "Insecure default configuration. Avoid setting secure:false, httpOnly:false, or sameSite:'none' for cookies or sessions."
        
      # Pattern 9: Lack of Proper Access Control
      - pattern: "router\\.(?:get|post|put|delete|patch)\\([^)]*?\\)\\s*(?!.*(?:authenticate|authorize|requireAuth|isAuthenticated|checkAuth|verifyToken|passport\\.authenticate))"
        location: "(?:routes|api|controllers)"
        message: "Potential lack of access control in route definition. Implement proper authentication and authorization middleware."
        
      # Pattern 10: Insecure File Operations
      - pattern: "(?:fs\\.(?:readFile|writeFile|appendFile|readdir|stat|access|open|unlink)|require)\\([^)]*?(?:(?:\\+|\\$\\{|\\`)[^)]*?(?:__dirname|__filename|process\\.cwd\\(\\)|path\\.(?:resolve|join)))"
        negative_pattern: "path\\.normalize|path\\.resolve|path\\.join"
        message: "Insecure file operations. Use path.normalize() and validate file paths to prevent directory traversal attacks."
        
      # Pattern 11: Lack of Proper Secrets Management
      - pattern: "(?:apiKey|secret|password|token|credentials)\\s*=\\s*(?:process\\.env\\.[A-Z_]+|config\\.[a-zA-Z0-9_]+|['\"][^'\"]+['\"])"
        negative_pattern: "(?:vault|secretsManager|keyVault|secretClient)"
        message: "Insecure secrets management. Consider using a dedicated secrets management solution instead of environment variables or configuration files."
        
      # Pattern 12: Insecure Randomness
      - pattern: "Math\\.random\\(\\)"
        location: "(?:auth|security|token|password|key|iv|nonce|salt)"
        message: "Insecure randomness. Use crypto.randomBytes() or a similar cryptographically secure random number generator for security-sensitive operations."
        
      # Pattern 13: Lack of Proper Input Sanitization for Templates
      - pattern: "(?:template|render|compile|ejs\\.render|handlebars\\.compile|pug\\.render)\\([^)]*?(?:(?:\\+|\\$\\{|\\`)[^)]*?(?:req\\.|request\\.|params\\.|query\\.|body\\.|user\\.|input\\.|form\\.))"
        message: "Potential template injection vulnerability. Sanitize user input before using in templates."
        
      # Pattern 14: Insecure WebSocket Implementation
      - pattern: "new\\s+WebSocket\\([^)]*?\\)|io\\.on\\(['\"]connection['\"]"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ivangrynenko/cursorrules](https://github.com/ivangrynenko/cursorrules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
