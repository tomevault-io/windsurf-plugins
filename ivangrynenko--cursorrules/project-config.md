---
trigger: always_on
description: Detect and prevent injection vulnerabilities in Python applications as defined in OWASP Top 10:2021-A03
---

 # Python Injection Security Standards (OWASP A03:2021)

This rule enforces security best practices to prevent injection vulnerabilities in Python applications, as defined in OWASP Top 10:2021-A03.

<rule>
name: python_injection
description: Detect and prevent injection vulnerabilities in Python applications as defined in OWASP Top 10:2021-A03
filters:
  - type: file_extension
    pattern: "\\.(py)$"
  - type: file_path
    pattern: ".*"

actions:
  - type: enforce
    conditions:
      # Pattern 1: SQL Injection - String concatenation in SQL queries
      - pattern: "cursor\\.(execute|executemany)\\([\"'][^\"']*\\s*[\\+%]|cursor\\.(execute|executemany)\\([^,]+\\+\\s*[a-zA-Z_][a-zA-Z0-9_]*"
        message: "Potential SQL injection vulnerability. Use parameterized queries with placeholders instead of string concatenation."
        
      # Pattern 2: SQL Injection - String formatting in SQL queries
      - pattern: "cursor\\.(execute|executemany)\\([\"'][^\"']*%[^\"']*[\"']\\s*%\\s*|cursor\\.(execute|executemany)\\([\"'][^\"']*{[^\"']*}[\"']\\.format"
        message: "Potential SQL injection vulnerability. Use parameterized queries with placeholders instead of string formatting."
        
      # Pattern 3: Command Injection - Shell command execution with user input
      - pattern: "(os\\.system|os\\.popen|subprocess\\.Popen|subprocess\\.call|subprocess\\.run|subprocess\\.check_output)\\([^)]*\\+\\s*[a-zA-Z_][a-zA-Z0-9_]*|\\b(os\\.system|os\\.popen|subprocess\\.Popen|subprocess\\.call|subprocess\\.run|subprocess\\.check_output)\\([^)]*format\\(|\\b(os\\.system|os\\.popen|subprocess\\.Popen|subprocess\\.call|subprocess\\.run|subprocess\\.check_output)\\([^)]*f['\"]"
        message: "Potential command injection vulnerability. Never use string concatenation or formatting with shell commands. Use subprocess with shell=False and pass arguments as a list."
        
      # Pattern 4: Command Injection - Shell=True in subprocess
      - pattern: "(subprocess\\.Popen|subprocess\\.call|subprocess\\.run|subprocess\\.check_output)\\([^)]*shell\\s*=\\s*True"
        message: "Using shell=True with subprocess functions is dangerous and can lead to command injection. Use shell=False (default) and pass arguments as a list."
        
      # Pattern 5: XSS - Unescaped template variables
      - pattern: "\\{\\{\\s*[^|]*\\s*\\}\\}|\\{\\%\\s*autoescape\\s+off\\s*\\%\\}"
        message: "Potential XSS vulnerability. Ensure all template variables are properly escaped. Avoid using 'autoescape off' in templates."
        
      # Pattern 6: XSS - Unsafe HTML rendering in Flask/Django
      - pattern: "render_template\\([^)]*\\)|render\\([^)]*\\)|mark_safe\\([^)]*\\)|safe\\s*\\|"
        message: "Potential XSS vulnerability. Ensure all user-supplied data is properly escaped before rendering in templates."
        
      # Pattern 7: Path Traversal - Unsafe file operations
      - pattern: "open\\([^)]*\\+|open\\([^)]*format\\(|open\\([^)]*f['\"]"
        message: "Potential path traversal vulnerability. Validate and sanitize file paths before opening files. Consider using os.path.abspath and os.path.normpath."
        
      # Pattern 8: LDAP Injection - Unsafe LDAP queries
      - pattern: "ldap\\.search\\([^)]*\\+|ldap\\.search\\([^)]*format\\(|ldap\\.search\\([^)]*f['\"]"
        message: "Potential LDAP injection vulnerability. Use proper LDAP escaping for user-supplied input in LDAP queries."
        
      # Pattern 9: NoSQL Injection - Unsafe MongoDB queries
      - pattern: "find\\(\\{[^}]*\\+|find\\(\\{[^}]*format\\(|find\\(\\{[^}]*f['\"]"
        message: "Potential NoSQL injection vulnerability. Use parameterized queries or proper escaping for MongoDB queries."
        
      # Pattern 10: Template Injection - Unsafe template rendering
      - pattern: "Template\\([^)]*\\)\\.(render|substitute)\\(|eval\\([^)]*\\)|exec\\([^)]*\\)"
        message: "Potential template injection or code injection vulnerability. Avoid using eval() or exec() with user input, and ensure template variables are properly validated."

  - type: suggest
    message: |
      **Python Injection Prevention Best Practices:**
      
      1. **SQL Injection Prevention:**
         - Use parameterized queries (prepared statements) with placeholders:
           ```python
           # Safe SQL query with parameters
           cursor.execute("SELECT * FROM users WHERE username = %s AND password = %s", (username, password))
           
           # Django ORM (safe by default)
           User.objects.filter(username=username, password=password)
           
           # SQLAlchemy (safe by default)
           session.query(User).filter(User.username == username, User.password == password)
           ```
         - Use ORM frameworks when possible (Django ORM, SQLAlchemy)
         - Apply proper input validation and sanitization
      
      2. **Command Injection Prevention:**
         - Never use shell=True with subprocess functions
         - Pass command arguments as a list, not a string:
           ```python
           # Safe command execution
           subprocess.run(["ls", "-l", user_dir], shell=False)
           ```
         - Use shlex.quote() if you must include user input in shell commands

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ivangrynenko/cursorrules](https://github.com/ivangrynenko/cursorrules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
