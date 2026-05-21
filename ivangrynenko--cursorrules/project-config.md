---
trigger: always_on
description: Detect and prevent security misconfigurations in Python applications as defined in OWASP Top 10:2021-A05
---

 # Python Security Misconfiguration Standards (OWASP A05:2021)

This rule enforces security best practices to prevent security misconfigurations in Python applications, as defined in OWASP Top 10:2021-A05.

<rule>
name: python_security_misconfiguration
description: Detect and prevent security misconfigurations in Python applications as defined in OWASP Top 10:2021-A05
filters:
  - type: file_extension
    pattern: "\\.(py|ini|cfg|yml|yaml|json|toml)$"
  - type: file_path
    pattern: ".*"

actions:
  - type: enforce
    conditions:
      # Pattern 1: Debug mode enabled in production settings
      - pattern: "DEBUG\\s*=\\s*True|debug\\s*=\\s*true|\"debug\"\\s*:\\s*true|debug:\\s*true"
        message: "Debug mode appears to be enabled. This should be disabled in production environments as it can expose sensitive information."
        
      # Pattern 2: Insecure cookie settings
      - pattern: "SESSION_COOKIE_SECURE\\s*=\\s*False|session_cookie_secure\\s*=\\s*false|\"session_cookie_secure\"\\s*:\\s*false|session_cookie_secure:\\s*false"
        message: "Insecure cookie configuration detected. Set SESSION_COOKIE_SECURE to True in production environments."
        
      # Pattern 3: Missing CSRF protection
      - pattern: "CSRF_ENABLED\\s*=\\s*False|csrf_enabled\\s*=\\s*false|\"csrf_enabled\"\\s*:\\s*false|csrf_enabled:\\s*false|WTF_CSRF_ENABLED\\s*=\\s*False"
        message: "CSRF protection appears to be disabled. Enable CSRF protection to prevent cross-site request forgery attacks."
        
      # Pattern 4: Insecure CORS settings
      - pattern: "CORS_ORIGIN_ALLOW_ALL\\s*=\\s*True|cors_origin_allow_all\\s*=\\s*true|\"cors_origin_allow_all\"\\s*:\\s*true|cors_origin_allow_all:\\s*true|Access-Control-Allow-Origin:\\s*\\*"
        message: "Overly permissive CORS configuration detected. Restrict CORS to specific origins rather than allowing all origins."
        
      # Pattern 5: Default or weak secret keys
      - pattern: "SECRET_KEY\\s*=\\s*['\"]default|SECRET_KEY\\s*=\\s*['\"][a-zA-Z0-9]{1,32}['\"]|secret_key\\s*=\\s*['\"]default|\"secret_key\"\\s*:\\s*\"default|secret_key:\\s*default"
        message: "Default or potentially weak secret key detected. Use a strong, randomly generated secret key and store it securely."
        
      # Pattern 6: Exposed sensitive information in error messages
      - pattern: "DEBUG_PROPAGATE_EXCEPTIONS\\s*=\\s*True|debug_propagate_exceptions\\s*=\\s*true|\"debug_propagate_exceptions\"\\s*:\\s*true|debug_propagate_exceptions:\\s*true"
        message: "Exception propagation in debug mode is enabled. This can expose sensitive information in error messages."
        
      # Pattern 7: Insecure SSL/TLS configuration
      - pattern: "SECURE_SSL_REDIRECT\\s*=\\s*False|secure_ssl_redirect\\s*=\\s*false|\"secure_ssl_redirect\"\\s*:\\s*false|secure_ssl_redirect:\\s*false"
        message: "SSL redirection appears to be disabled. Enable SSL redirection to ensure secure communications."
        
      # Pattern 8: Missing security headers
      - pattern: "SECURE_HSTS_SECONDS\\s*=\\s*0|secure_hsts_seconds\\s*=\\s*0|\"secure_hsts_seconds\"\\s*:\\s*0|secure_hsts_seconds:\\s*0"
        message: "HTTP Strict Transport Security (HSTS) appears to be disabled. Enable HSTS to enforce secure communications."
        
      # Pattern 9: Exposed sensitive directories
      - pattern: "@app\\.route\\(['\"]/(admin|console|management|config|settings|system)['\"]"
        message: "Potentially sensitive endpoint exposed without access controls. Ensure proper authentication and authorization for administrative endpoints."
        
      # Pattern 10: Default accounts or credentials
      - pattern: "username\\s*=\\s*['\"]admin['\"]|password\\s*=\\s*['\"]admin|password\\s*=\\s*['\"]password|password\\s*=\\s*['\"]123|user\\s*=\\s*['\"]root['\"]"
        message: "Default or weak credentials detected. Never use default or easily guessable credentials in any environment."
        
      # Pattern 11: Insecure file permissions
      - pattern: "os\\.chmod\\([^,]+,\\s*0o777\\)|os\\.chmod\\([^,]+,\\s*777\\)"
        message: "Overly permissive file permissions detected. Use the principle of least privilege for file permissions."
        
      # Pattern 12: Exposed version information
      - pattern: "@app\\.route\\(['\"]/(version|build|status|health)['\"]"
        message: "Endpoints that may expose version information detected. Ensure these endpoints don't reveal sensitive details about your application."
        
      # Pattern 13: Insecure deserialization
      - pattern: "pickle\\.loads|yaml\\.load\\([^,)]+\\)|json\\.loads\\([^,)]+,\\s*[^)]*object_hook"
        message: "Potentially insecure deserialization detected. Use safer alternatives like yaml.safe_load() or validate input before deserialization."
        
      # Pattern 14: Missing timeout settings
      - pattern: "requests\\.get\\([^,)]+\\)|requests\\.(post|put|delete|patch)\\([^,)]+\\)"
        message: "HTTP request without timeout setting detected. Always set timeouts for HTTP requests to prevent denial of service."
        
      # Pattern 15: Insecure upload directory

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ivangrynenko/cursorrules](https://github.com/ivangrynenko/cursorrules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
