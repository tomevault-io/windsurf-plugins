---
trigger: always_on
description: Detect and prevent identification and authentication failures in Python applications as defined in OWASP Top 10:2021-A07
---

 # Python Identification and Authentication Failures Standards (OWASP A07:2021)

This rule enforces security best practices to prevent identification and authentication failures in Python applications, as defined in OWASP Top 10:2021-A07.

<rule>
name: python_authentication_failures
description: Detect and prevent identification and authentication failures in Python applications as defined in OWASP Top 10:2021-A07
filters:
  - type: file_extension
    pattern: "\\.(py|ini|cfg|yml|yaml|json|toml)$"
  - type: file_path
    pattern: ".*"

actions:
  - type: enforce
    conditions:
      # Pattern 1: Weak password validation
      - pattern: "password\\s*=\\s*['\"][^'\"]{1,7}['\"]|min_length\\s*=\\s*[1-7]"
        message: "Weak password policy detected. Passwords should be at least 8 characters long and include complexity requirements."
        
      # Pattern 2: Hardcoded credentials
      - pattern: "(username|user|login|password|passwd|pwd|secret|api_key|apikey|token)\\s*=\\s*['\"][^'\"]+['\"]"
        message: "Hardcoded credentials detected. Store sensitive credentials in environment variables or a secure vault."
        
      # Pattern 3: Missing password hashing
      - pattern: "password\\s*=\\s*request\\.form\\[\\'password\\'\\]|password\\s*=\\s*request\\.POST\\.get\\(\\'password\\'\\)"
        message: "Storing or comparing plain text passwords detected. Always hash passwords before storage or comparison."
        
      # Pattern 4: Insecure password hashing
      - pattern: "hashlib\\.md5\\(|hashlib\\.sha1\\(|hashlib\\.sha224\\("
        message: "Insecure hashing algorithm detected. Use strong hashing algorithms like bcrypt, Argon2, or PBKDF2."
        
      # Pattern 5: Missing brute force protection
      - pattern: "@app\\.route\\(['\"]\\/(login|signin|authenticate)['\"]"
        message: "Authentication endpoint detected without rate limiting or brute force protection. Implement account lockout or rate limiting."
        
      # Pattern 6: Insecure session management
      - pattern: "session\\[\\'user_id\\'\\]\\s*=|session\\[\\'authenticated\\'\\]\\s*=\\s*True"
        message: "Session management detected. Ensure proper session security with secure cookies, proper expiration, and rotation."
        
      # Pattern 7: Missing CSRF protection in authentication
      - pattern: "form\\s*=\\s*FlaskForm|class\\s+\\w+Form\\(\\s*FlaskForm\\s*\\)|class\\s+\\w+Form\\(\\s*Form\\s*\\)"
        message: "Form handling detected. Ensure CSRF protection is enabled for all authentication forms."
        
      # Pattern 8: Insecure remember me functionality
      - pattern: "remember_me|remember_token|stay_logged_in"
        message: "Remember me functionality detected. Ensure secure implementation with proper expiration and refresh mechanisms."
        
      # Pattern 9: Insecure password reset
      - pattern: "@app\\.route\\(['\"]\\/(reset-password|forgot-password|recover)['\"]"
        message: "Password reset functionality detected. Ensure secure implementation with time-limited tokens and proper user verification."
        
      # Pattern 10: Missing multi-factor authentication
      - pattern: "def\\s+login|def\\s+authenticate|def\\s+signin"
        message: "Authentication function detected. Consider implementing multi-factor authentication for sensitive operations."
        
      # Pattern 11: Insecure direct object reference in user management
      - pattern: "User\\.objects\\.get\\(id=|User\\.query\\.get\\(|get_user_by_id\\("
        message: "Direct user lookup detected. Ensure proper authorization checks before accessing user data."
        
      # Pattern 12: Insecure JWT implementation
      - pattern: "jwt\\.encode\\(|jwt\\.decode\\("
        message: "JWT usage detected. Ensure proper signing, validation, expiration, and refresh mechanisms for JWTs."
        
      # Pattern 13: Missing secure flag in cookies
      - pattern: "set_cookie\\([^,]+,[^,]+,[^,]*secure=False|set_cookie\\([^,]+,[^,]+(?!,\\s*secure=True)"
        message: "Cookie setting without secure flag detected. Set secure=True for all authentication cookies."
        
      # Pattern 14: Missing HTTP-only flag in cookies
      - pattern: "set_cookie\\([^,]+,[^,]+,[^,]*httponly=False|set_cookie\\([^,]+,[^,]+(?!,\\s*httponly=True)"
        message: "Cookie setting without httponly flag detected. Set httponly=True for all authentication cookies."
        
      # Pattern 15: Insecure default credentials
      - pattern: "DEFAULT_USERNAME|DEFAULT_PASSWORD|ADMIN_USERNAME|ADMIN_PASSWORD"
        message: "Default credential configuration detected. Remove default credentials from production code."

  - type: suggest
    message: |
      **Python Authentication Security Best Practices:**
      
      1. **Password Storage:**
         - Use strong hashing algorithms with salting
         - Implement proper work factors
         - Example with passlib:
           ```python
           from passlib.hash import argon2
           
           # Hash a password
           hashed_password = argon2.hash("user_password")
           
           # Verify a password
           is_valid = argon2.verify("user_password", hashed_password)
           ```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ivangrynenko/cursorrules](https://github.com/ivangrynenko/cursorrules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
