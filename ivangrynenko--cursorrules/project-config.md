---
trigger: always_on
description: Detect and prevent vulnerabilities related to outdated dependencies and components in Python applications as defined in OWASP Top 10:2021-A06
---

 # Python Vulnerable and Outdated Components Standards (OWASP A06:2021)

This rule enforces security best practices to prevent vulnerabilities related to outdated dependencies and components in Python applications, as defined in OWASP Top 10:2021-A06.

<rule>
name: python_vulnerable_outdated_components
description: Detect and prevent vulnerabilities related to outdated dependencies and components in Python applications as defined in OWASP Top 10:2021-A06
filters:
  - type: file_extension
    pattern: "\\.(py|txt|ini|cfg|yml|yaml|json|toml)$"
  - type: file_path
    pattern: ".*"

actions:
  - type: enforce
    conditions:
      # Pattern 1: Unpinned dependencies in requirements files
      - pattern: "^(django|flask|fastapi|requests|cryptography|pyyaml|sqlalchemy|celery|numpy|pandas|pillow|tensorflow|torch|boto3|psycopg2)\\s*$"
        file_pattern: "requirements.*\\.txt$|setup\\.py$|pyproject\\.toml$"
        message: "Unpinned dependency detected. Always pin dependencies to specific versions to prevent automatic updates to potentially vulnerable versions."
        
      # Pattern 2: Outdated/vulnerable Django versions
      - pattern: "django([<>=]=|~=|==)\\s*[\"']?(1\\.|2\\.[0-2]\\.|3\\.[0-2]\\.|4\\.0\\.)[0-9]+[\"']?"
        message: "Potentially outdated Django version detected. Consider upgrading to the latest stable version with security updates."
        
      # Pattern 3: Outdated/vulnerable Flask versions
      - pattern: "flask([<>=]=|~=|==)\\s*[\"']?(0\\.|1\\.[0-3]\\.|2\\.0\\.[0-3])[0-9]*[\"']?"
        message: "Potentially outdated Flask version detected. Consider upgrading to the latest stable version with security updates."
        
      # Pattern 4: Outdated/vulnerable Requests versions
      - pattern: "requests([<>=]=|~=|==)\\s*[\"']?(0\\.|1\\.|2\\.[0-2][0-5]\\.[0-9]+)[\"']?"
        message: "Potentially outdated Requests version detected. Consider upgrading to the latest stable version with security updates."
        
      # Pattern 5: Outdated/vulnerable Cryptography versions
      - pattern: "cryptography([<>=]=|~=|==)\\s*[\"']?(0\\.|1\\.|2\\.|3\\.[0-3]\\.|3\\.4\\.[0-7])[0-9]*[\"']?"
        message: "Potentially outdated Cryptography version detected. Consider upgrading to the latest stable version with security updates."
        
      # Pattern 6: Outdated/vulnerable PyYAML versions
      - pattern: "pyyaml([<>=]=|~=|==)\\s*[\"']?(0\\.|1\\.|2\\.|3\\.|4\\.|5\\.[0-5]\\.[0-9]+)[\"']?"
        message: "Potentially outdated PyYAML version detected. Consider upgrading to the latest stable version with security updates."
        
      # Pattern 7: Outdated/vulnerable Pillow versions
      - pattern: "pillow([<>=]=|~=|==)\\s*[\"']?(0\\.|1\\.|2\\.|3\\.|4\\.|5\\.|6\\.|7\\.|8\\.[0-3]\\.[0-9]+)[\"']?"
        message: "Potentially outdated Pillow version detected. Consider upgrading to the latest stable version with security updates."
        
      # Pattern 8: Direct imports of deprecated modules
      - pattern: "from\\s+xml\\.etree\\.ElementTree\\s+import\\s+.*parse|from\\s+urllib2\\s+import|from\\s+urllib\\s+import\\s+urlopen|import\\s+cgi|import\\s+imp"
        message: "Use of deprecated or insecure module detected. Consider using more secure alternatives."
        
      # Pattern 9: Use of deprecated functions
      - pattern: "\\.set_password\\([^)]*\\)|hashlib\\.md5\\(|hashlib\\.sha1\\(|random\\.random\\(|random\\.randrange\\(|random\\.randint\\("
        message: "Use of deprecated or insecure function detected. Consider using more secure alternatives."
        
      # Pattern 10: Insecure dependency loading
      - pattern: "__import__\\(|importlib\\.import_module\\(|exec\\(|eval\\("
        message: "Dynamic code execution or module loading detected. This can lead to code injection if user input is involved."
        
      # Pattern 11: Outdated TLS/SSL versions
      - pattern: "ssl\\.PROTOCOL_TLSv1|ssl\\.PROTOCOL_TLSv1_1|ssl\\.PROTOCOL_SSLv2|ssl\\.PROTOCOL_SSLv3|ssl\\.PROTOCOL_TLSv1_2"
        message: "Outdated TLS/SSL protocol version detected. Use ssl.PROTOCOL_TLS_CLIENT or ssl.PROTOCOL_TLS_SERVER instead."
        
      # Pattern 12: Insecure deserialization libraries
      - pattern: "import\\s+pickle|import\\s+marshal|import\\s+shelve"
        message: "Use of potentially insecure deserialization library detected. Ensure these are not used with untrusted data."
        
      # Pattern 13: Outdated/vulnerable SQLAlchemy versions
      - pattern: "sqlalchemy([<>=]=|~=|==)\\s*[\"']?(0\\.|1\\.[0-3]\\.[0-9]+)[\"']?"
        message: "Potentially outdated SQLAlchemy version detected. Consider upgrading to the latest stable version with security updates."
        
      # Pattern 14: Outdated/vulnerable Celery versions
      - pattern: "celery([<>=]=|~=|==)\\s*[\"']?(0\\.|1\\.|2\\.|3\\.|4\\.[0-4]\\.[0-9]+)[\"']?"
        message: "Potentially outdated Celery version detected. Consider upgrading to the latest stable version with security updates."
        
      # Pattern 15: Insecure package installation
      - pattern: "pip\\s+install\\s+.*--no-deps|pip\\s+install\\s+.*--user|pip\\s+install\\s+.*--pre|pip\\s+install\\s+.*--index-url\\s+http://"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ivangrynenko/cursorrules](https://github.com/ivangrynenko/cursorrules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
