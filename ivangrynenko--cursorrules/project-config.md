---
trigger: always_on
description: Detect and prevent security logging and monitoring failures in JavaScript applications as defined in OWASP Top 10:2021-A09
---

# JavaScript Security Logging and Monitoring Failures (OWASP A09:2021)

<rule>
name: javascript_security_logging_monitoring_failures
description: Detect and prevent security logging and monitoring failures in JavaScript applications as defined in OWASP Top 10:2021-A09

actions:
  - type: enforce
    conditions:
      # Pattern 1: Missing Error Logging
      - pattern: "(?:try\\s*{[^}]*}\\s*catch\\s*\\([^)]*\\)\\s*{[^}]*})(?![^;{]*(?:console\\.(?:error|warn|log)|logger?\\.(?:error|warn|log)|captureException))"
        message: "Error caught without proper logging. Implement structured error logging for security events."
        
      # Pattern 2: Sensitive Data in Logs
      - pattern: "console\\.(?:log|warn|error|info|debug)\\s*\\([^)]*(?:password|token|secret|key|credential|auth|jwt|session|cookie)"
        negative_pattern: "\\*\\*\\*|redact|mask|sanitize"
        message: "Potential sensitive data in logs. Ensure sensitive information is redacted before logging."
        
      # Pattern 3: Missing Authentication Logging
      - pattern: "(?:login|signin|authenticate|auth)\\s*\\([^)]*\\)\\s*{[^}]*}"
        negative_pattern: "(?:log|audit|record|track)\\s*\\("
        message: "Authentication function without logging. Log authentication attempts, successes, and failures."
        
      # Pattern 4: Missing Authorization Logging
      - pattern: "(?:authorize|checkPermission|hasAccess|isAuthorized|can)\\s*\\([^)]*\\)\\s*{[^}]*}"
        negative_pattern: "(?:log|audit|record|track)\\s*\\("
        message: "Authorization check without logging. Log access control decisions, especially denials."
        
      # Pattern 5: Insufficient Error Detail
      - pattern: "(?:console\\.error|logger?\\.error)\\s*\\([^)]*(?:error|err|exception)\\s*\\)"
        negative_pattern: "(?:error\\.(?:message|stack|code|name)|JSON\\.stringify\\(error\\)|serialize)"
        message: "Error logging with insufficient detail. Include error type, message, stack trace, and context."
        
      # Pattern 6: Missing Security Event Logging
      - pattern: "(?:bruteForce|rateLimit|block|blacklist|suspicious|anomaly|threat|attack|intrusion|malicious)"
        negative_pattern: "(?:log|audit|record|track|monitor|alert|notify)"
        message: "Security event detection without logging. Implement logging for all security-relevant events."
        
      # Pattern 7: Inconsistent Log Formats
      - pattern: "console\\.(?:log|warn|error|info|debug)\\s*\\("
        negative_pattern: "JSON\\.stringify|structured|format"
        message: "Inconsistent log format. Use structured logging with consistent formats for easier analysis."
        
      # Pattern 8: Missing Log Correlation ID
      - pattern: "(?:api|http|fetch|axios|request)\\s*\\([^)]*\\)"
        negative_pattern: "(?:correlationId|requestId|traceId|spanId|context)"
        message: "API request without correlation ID. Include correlation IDs in logs for request tracing."
        
      # Pattern 9: Missing High-Value Transaction Logging
      - pattern: "(?:payment|transaction|order|purchase|transfer|withdraw|deposit)\\s*\\([^)]*\\)"
        negative_pattern: "(?:log|audit|record|track)"
        message: "High-value transaction without audit logging. Implement comprehensive logging for all transactions."
        
      # Pattern 10: Client-Side Logging Issues
      - pattern: "(?:window\\.onerror|window\\.addEventListener\\s*\\(\\s*['\"]error['\"])"
        negative_pattern: "(?:send|report|log|capture|track)"
        message: "Client-side error handler without reporting. Implement error reporting to backend services."
        
      # Pattern 11: Missing Log Levels
      - pattern: "console\\.log\\s*\\("
        negative_pattern: "logger?\\.(?:error|warn|info|debug|trace)"
        message: "Using console.log without proper log levels. Implement a logging library with appropriate log levels."
        
      # Pattern 12: Missing Monitoring Integration
      - pattern: "package\\.json"
        negative_pattern: "(?:sentry|newrelic|datadog|appinsights|loggly|splunk|elasticsearch|winston|bunyan|pino|loglevel)"
        file_pattern: "package\\.json$"
        message: "No logging or monitoring dependencies detected. Consider adding a proper logging library and monitoring integration."
        
      # Pattern 13: Missing Log Aggregation
      - pattern: "(?:docker-compose\\.ya?ml|\\.env|\\.env\\.example|Dockerfile)"
        negative_pattern: "(?:sentry|newrelic|datadog|appinsights|loggly|splunk|elasticsearch|logstash|fluentd|kibana)"
        file_pattern: "(?:docker-compose\\.ya?ml|\\.env|\\.env\\.example|Dockerfile)$"
        message: "No log aggregation service configured. Implement centralized log collection and analysis."
        
      # Pattern 14: Missing Health Checks
      - pattern: "(?:express|koa|fastify|hapi|http\\.createServer)"
        negative_pattern: "(?:health|status|heartbeat|alive|ready)"
        message: "Server without health check endpoint. Implement health checks for monitoring service status."
        
      # Pattern 15: Missing Rate Limiting Logs
      - pattern: "(?:rateLimit|throttle|limiter)"
        negative_pattern: "(?:log|record|track|monitor|alert|notify)"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ivangrynenko/cursorrules](https://github.com/ivangrynenko/cursorrules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
