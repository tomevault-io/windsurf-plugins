---
trigger: always_on
description: Detect and prevent insecure design patterns in Python applications as defined in OWASP Top 10:2021-A04
---

 # Python Insecure Design Security Standards (OWASP A04:2021)

This rule enforces security best practices to prevent insecure design vulnerabilities in Python applications, as defined in OWASP Top 10:2021-A04.

<rule>
name: python_insecure_design
description: Detect and prevent insecure design patterns in Python applications as defined in OWASP Top 10:2021-A04
filters:
  - type: file_extension
    pattern: "\\.(py)$"
  - type: file_path
    pattern: ".*"

actions:
  - type: enforce
    conditions:
      # Pattern 1: Lack of input validation
      - pattern: "def\\s+[a-zA-Z0-9_]+\\([^)]*\\):\\s*(?![^#]*validate|[^#]*clean|[^#]*sanitize|[^#]*check|[^#]*is_valid)"
        message: "Function lacks input validation. Consider implementing validation for all user-supplied inputs."
        
      # Pattern 2: Hardcoded business rules
      - pattern: "if\\s+[a-zA-Z0-9_]+\\s*(==|!=|>|<|>=|<=)\\s*['\"][^'\"]+['\"]:"
        message: "Hardcoded business rules detected. Consider using configuration files or database-driven rules for better maintainability."
        
      # Pattern 3: Lack of rate limiting
      - pattern: "@(app|api|route|blueprint)\\.(get|post|put|delete|patch)\\([^)]*\\)\\s*\\n\\s*(?![^#]*rate_limit|[^#]*throttle|[^#]*limiter)"
        message: "API endpoint lacks rate limiting. Consider implementing rate limiting to prevent abuse."
        
      # Pattern 4: Insecure default configurations
      - pattern: "DEBUG\\s*=\\s*True|DEVELOPMENT\\s*=\\s*True|TESTING\\s*=\\s*True"
        message: "Insecure default configuration detected. Ensure debug/development modes are disabled in production."
        
      # Pattern 5: Lack of error handling
      - pattern: "(?<!try:\\s*\\n)[^#]*\\n\\s*(?!except|finally)"
        message: "Consider implementing proper error handling with try-except blocks for operations that might fail."
        
      # Pattern 6: Insecure direct object references
      - pattern: "get_object_or_404\\(\\s*[^,]+,\\s*pk\\s*=\\s*request\\.(GET|POST|args|form|json)\\[['\"][^'\"]+['\"]\\]\\s*\\)|get\\(\\s*id\\s*=\\s*request\\.(GET|POST|args|form|json)"
        message: "Potential insecure direct object reference. Validate user's permission to access the requested object."
        
      # Pattern 7: Missing authentication checks
      - pattern: "@(app|api|route|blueprint)\\.(get|post|put|delete|patch)\\([^)]*\\)\\s*\\n\\s*(?!.*@login_required|.*@auth\\.login_required|.*@jwt_required|.*current_user|.*request\\.user)"
        message: "Endpoint lacks authentication checks. Consider adding authentication requirements for sensitive operations."
        
      # Pattern 8: Lack of proper logging
      - pattern: "except\\s+[a-zA-Z0-9_]+\\s*(?:as\\s+[a-zA-Z0-9_]+)?:\\s*(?!.*logger\\.|.*logging\\.|.*print)"
        message: "Exception caught without proper logging. Implement proper logging for exceptions to aid in debugging and monitoring."
        
      # Pattern 9: Insecure file uploads
      - pattern: "request\\.files\\[['\"][^'\"]+['\"]\\]|FileField\\(|FileStorage\\("
        message: "File upload functionality detected. Ensure proper validation of file types, sizes, and implement virus scanning if applicable."
        
      # Pattern 10: Lack of security headers
      - pattern: "response\\.(headers|set_header)\\([^)]*\\)|return\\s+Response\\([^)]*\\)|return\\s+make_response\\([^)]*\\)"
        message: "Consider adding security headers (Content-Security-Policy, X-Content-Type-Options, etc.) to HTTP responses."

  - type: suggest
    message: |
      **Python Secure Design Best Practices:**
      
      1. **Implement Defense in Depth:**
         - Layer security controls throughout your application
         - Don't rely on a single security mechanism
         - Assume that each security layer can be bypassed
      
      2. **Use Secure Defaults:**
         - Start with secure configurations by default
         - Require explicit opt-in for less secure options
         - Example for Flask:
           ```python
           app.config.update(
               SESSION_COOKIE_SECURE=True,
               SESSION_COOKIE_HTTPONLY=True,
               SESSION_COOKIE_SAMESITE='Lax',
               PERMANENT_SESSION_LIFETIME=timedelta(hours=1)
           )
           ```
      
      3. **Implement Proper Access Control:**
         - Use role-based access control (RBAC)
         - Implement principle of least privilege
         - Validate access at the controller and service layers
         - Example:
           ```python
           @app.route('/admin')
           @roles_required('admin')  # Using Flask-Security
           def admin_dashboard():
               return render_template('admin/dashboard.html')
           ```
      
      4. **Use Rate Limiting:**
         - Protect against brute force and DoS attacks
         - Example with Flask-Limiter:
           ```python
           from flask_limiter import Limiter
           limiter = Limiter(app)
           
           @app.route('/login', methods=['POST'])
           @limiter.limit("5 per minute")
           def login():
               # Login logic
           ```
      
      5. **Implement Proper Error Handling:**
         - Catch and log exceptions appropriately

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ivangrynenko/cursorrules](https://github.com/ivangrynenko/cursorrules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
