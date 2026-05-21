---
trigger: always_on
description: Detect and prevent security logging and monitoring failures in Python applications as defined in OWASP Top 10:2021-A09
---

 # Python Security Logging and Monitoring Failures Standards (OWASP A09:2021)

This rule enforces security best practices to prevent security logging and monitoring failures in Python applications, as defined in OWASP Top 10:2021-A09.

<rule>
name: python_logging_monitoring_failures
description: Detect and prevent security logging and monitoring failures in Python applications as defined in OWASP Top 10:2021-A09
filters:
  - type: file_extension
    pattern: "\\.(py|ini|cfg|yml|yaml|json|toml)$"
  - type: file_path
    pattern: ".*"

actions:
  - type: enforce
    conditions:
      # Pattern 1: Missing logging in authentication functions
      - pattern: "def\\s+(login|authenticate|signin|logout|signout).*?:[^\\n]*?(?!.*logging\\.(info|warning|error|critical))"
        message: "Authentication function without logging detected. Always log authentication events, especially failures, for security monitoring."
        
      # Pattern 2: Missing logging in authorization functions
      - pattern: "def\\s+(authorize|check_permission|has_permission|is_authorized|require_permission).*?:[^\\n]*?(?!.*logging\\.(info|warning|error|critical))"
        message: "Authorization function without logging detected. Always log authorization decisions, especially denials, for security monitoring."
        
      # Pattern 3: Missing logging in security-sensitive operations
      - pattern: "def\\s+(create_user|update_user|delete_user|reset_password|change_password).*?:[^\\n]*?(?!.*logging\\.(info|warning|error|critical))"
        message: "Security-sensitive user operation without logging detected. Always log security-sensitive operations for audit trails."
        
      # Pattern 4: Missing logging in exception handlers
      - pattern: "except\\s+[^:]+:[^\\n]*?(?!.*logging\\.(warning|error|critical|exception))"
        message: "Exception handler without logging detected. Always log exceptions, especially in security-sensitive code, for monitoring and debugging."
        
      # Pattern 5: Logging sensitive data
      - pattern: "logging\\.(debug|info|warning|error|critical)\\([^)]*?(password|token|secret|key|credential|auth)"
        message: "Potential sensitive data logging detected. Avoid logging sensitive information like passwords, tokens, or keys."
        
      # Pattern 6: Insufficient log level in security context
      - pattern: "logging\\.debug\\([^)]*?(auth|login|permission|security|attack|hack|exploit|vulnerability)"
        message: "Debug-level logging for security events detected. Use appropriate log levels (INFO, WARNING, ERROR) for security events."
        
      # Pattern 7: Missing logging configuration
      - pattern: "import\\s+logging(?!.*logging\\.basicConfig|.*logging\\.config)"
        message: "Logging import without configuration detected. Configure logging properly with appropriate handlers, formatters, and levels."
        
      # Pattern 8: Insecure logging configuration
      - pattern: "logging\\.basicConfig\\([^)]*?level\\s*=\\s*logging\\.DEBUG"
        message: "Debug-level logging configuration detected. Use appropriate log levels in production to avoid excessive logging."
        
      # Pattern 9: Missing request/response logging in web frameworks
      - pattern: "@app\\.route\\(['\"][^'\"]+['\"]|@api_view\\(|class\\s+\\w+\\(APIView\\)|class\\s+\\w+\\(View\\)"
        message: "Web endpoint without request logging detected. Consider logging requests and responses for security monitoring."
        
      # Pattern 10: Missing correlation IDs in logs
      - pattern: "logging\\.(debug|info|warning|error|critical)\\([^)]*?(?!.*request_id|.*correlation_id|.*trace_id)"
        message: "Logging without correlation ID detected. Include correlation IDs in logs to trace requests across systems."
        
      # Pattern 11: Missing error handling for logging failures
      - pattern: "logging\\.(debug|info|warning|error|critical)\\([^)]*?\\)"
        message: "Logging without error handling detected. Handle potential logging failures to ensure critical events are not missed."
        
      # Pattern 12: Missing logging for database operations
      - pattern: "(execute|executemany|cursor\\.execute|session\\.execute|query)\\([^)]*?(?!.*logging\\.(debug|info|warning|error|critical))"
        message: "Database operation without logging detected. Consider logging database operations for audit trails and security monitoring."
        
      # Pattern 13: Missing logging for file operations
      - pattern: "open\\([^)]+,\\s*['\"]w['\"]|open\\([^)]+,\\s*['\"]a['\"]|write\\(|writelines\\("
        message: "File write operation without logging detected. Consider logging file operations for audit trails."
        
      # Pattern 14: Missing logging for subprocess execution
      - pattern: "subprocess\\.(call|run|Popen)\\([^)]*?(?!.*logging\\.(debug|info|warning|error|critical))"
        message: "Subprocess execution without logging detected. Always log command execution for security monitoring."
        
      # Pattern 15: Missing centralized logging configuration
      - pattern: "logging\\.basicConfig\\([^)]*?(?!.*filename|.*handlers)"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ivangrynenko/cursorrules](https://github.com/ivangrynenko/cursorrules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
