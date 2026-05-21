---
trigger: always_on
description: Detect and prevent broken access control patterns in JavaScript applications as defined in OWASP Top 10:2021-A01
---

# JavaScript Broken Access Control (OWASP A01:2021)

This rule identifies and prevents broken access control vulnerabilities in JavaScript applications, focusing on both browser and Node.js environments, as defined in OWASP Top 10:2021-A01.

<rule>
name: javascript_broken_access_control
description: Detect and prevent broken access control patterns in JavaScript applications as defined in OWASP Top 10:2021-A01

actions:
  - type: enforce
    conditions:
      # Pattern 1: Detect Direct Reference to User-Supplied IDs (IDOR vulnerability)
      - pattern: "(?:req|request)\\.(?:params|query|body)\\.(?:id|userId|recordId)[^\\n]*?(?:findById|getById|find\\(|get\\()"
        message: "Potential Insecure Direct Object Reference (IDOR) vulnerability. User-supplied IDs should be validated against user permissions before database access."
        
      # Pattern 2: Detect Missing Authorization Checks in Route Handlers
      - pattern: "(?:app|router)\\.(?:get|post|put|delete|patch)\\(['\"][^'\"]+['\"],\\s*(?:async)?\\s*\\(?(?:req|request),\\s*(?:res|response)(?:,[^\\)]+)?\\)?\\s*=>\\s*\\{[^\\}]*?\\}\\)"
        negative_pattern: "(?:isAuthenticated|isAuthorized|checkPermission|verifyAccess|auth\\.check|authenticate|authorize|userHasAccess|checkAuth|permissions\\.|requireAuth|requiresAuth|ensureAuth|\\bauth\\b|\\broles?\\b|\\bpermission\\b|\\baccess\\b)"
        message: "Route handler appears to be missing authorization checks. Implement proper access control to verify user permissions before processing requests."
        
      # Pattern 3: Detect JWT Token Validation Issues
      - pattern: "(?:jwt|jsonwebtoken)\\.verify\\((?:[^,]+),\\s*['\"]((?:[A-Za-z0-9+/]{4})*(?:[A-Za-z0-9+/]{2}==|[A-Za-z0-9+/]{3}=)?)['\"]"
        message: "Hardcoded JWT secret detected. Store JWT secrets securely in environment variables or a configuration manager."
        
      # Pattern 4: Detect Client-Side Authorization Checks
      - pattern: "if\\s*\\((?:user|currentUser)\\.(?:role|isAdmin|hasPermission|can[A-Z][a-zA-Z]+|is[A-Z][a-zA-Z]+)\\)\\s*\\{[^\\}]*?(?:fetch|axios|\\$\\.ajax|http\\.get|http\\.post)\\([^\\)]*?\\)"
        message: "Authorization logic implemented on client-side. Client-side authorization checks can be bypassed. Always enforce authorization on the server."
        
      # Pattern 5: Detect Improper CORS Configuration
      - pattern: "(?:app\\.use\\(cors\\(\\{[^\\}]*?origin:\\s*['\"]\\*['\"])|Access-Control-Allow-Origin:\\s*['\"]\\*['\"]"
        message: "Wildcard CORS policy detected. This allows any domain to make cross-origin requests. Restrict CORS to specific trusted domains."
        
      # Pattern 6: Detect Lack of Role Checks in Admin Functions
      - pattern: "(?:function|const)\\s+(?:admin|updateUser|deleteUser|createUser|updateRole|manageUsers|setPermission)[^\\{]*?\\{[^\\}]*?\\}"
        negative_pattern: "(?:role|permission|isAdmin|hasAccess|authorize|authenticate|auth\\.check|checkPermission|checkRole|verifyRole|ensureAdmin|adminOnly|adminRequired|requirePermission)"
        message: "Administrative function appears to be missing role or permission checks. Implement proper authorization checks to restrict access to administrative functions."
        
      # Pattern 7: Detect Missing Login Rate Limiting
      - pattern: "(?:function|const)\\s+(?:login|signin|authenticate|auth)[^\\{]*?\\{[^\\}]*?(?:compare(?:Sync)?|check(?:Password)?|match(?:Password)?|verify(?:Password)?)[^\\}]*?\\}"
        negative_pattern: "(?:rate(?:Limit)?|throttle|limit|delay|cooldown|attempt|counter|maxTries|maxAttempts|lockout|timeout)"
        message: "Login function appears to be missing rate limiting. Implement rate limiting to prevent brute force attacks."
        
      # Pattern 8: Detect Horizontal Privilege Escalation Vulnerability
      - pattern: "(?:findById|findOne|findByPk|get)\\((?:req|request)\\.(?:params|query|body)\\.(?:id|userId|accountId)\\)"
        negative_pattern: "(?:!=|!==|===|==)\\s*(?:req\\.user\\.id|req\\.userId|currentUser\\.id|user\\.id|session\\.userId)"
        message: "Potential horizontal privilege escalation vulnerability. Ensure the requested resource belongs to the authenticated user."
        
      # Pattern 9: Detect Missing CSRF Protection
      - pattern: "(?:app|router)\\.(?:post|put|delete|patch)\\(['\"][^'\"]+['\"]"
        negative_pattern: "(?:csrf|xsrf|csurf|csrfProtection|antiForgery|csrfToken|csrfMiddleware)"
        message: "Route may be missing CSRF protection. Implement CSRF tokens for state-changing operations to prevent cross-site request forgery attacks."
        
      # Pattern 10: Detect Bypassing Access Control with Path Traversal
      - pattern: "(?:fs|require)(?:\\.promises)?\\.(read|open|access|stat)(?:File|Sync)?\\([^\\)]*?(?:req|request)\\.(?:params|query|body|path)\\.[^\\)]*?\\)"
        negative_pattern: "(?:normalize|resolve|sanitize|validate|pathValidation|checkPath)"
        message: "Potential path traversal vulnerability in file access. Validate and sanitize user-supplied paths to prevent directory traversal attacks."
        
      # Pattern 11: Detect Missing Authentication Middleware
      - pattern: "(?:new\\s+)?express\\(\\)|(?:import|require)\\(['\"]express['\"]\\)"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ivangrynenko/cursorrules](https://github.com/ivangrynenko/cursorrules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
