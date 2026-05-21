---
trigger: always_on
description: Detect and prevent software and data integrity failures in Python applications as defined in OWASP Top 10:2021-A08
---

# Python Software and Data Integrity Failures Standards (OWASP A08:2021)

This rule enforces security best practices to prevent software and data integrity failures in Python applications, as defined in OWASP Top 10:2021-A08.

<rule>
name: python_integrity_failures
description: Detect and prevent software and data integrity failures in Python applications as defined in OWASP Top 10:2021-A08
filters:
  - type: file_extension
    pattern: "\\.(py|ini|cfg|yml|yaml|json|toml)$"
  - type: file_path
    pattern: ".*"

actions:
  - type: enforce
    conditions:
      # Pattern 1: Insecure deserialization with pickle
      - pattern: "pickle\\.loads\\(|pickle\\.load\\(|cPickle\\.loads\\(|cPickle\\.load\\("
        message: "Insecure deserialization detected with pickle. Pickle is not secure against maliciously constructed data and should not be used with untrusted input."
        
      # Pattern 2: Insecure deserialization with yaml.load
      - pattern: "yaml\\.load\\([^,)]+\\)|yaml\\.load\\([^,)]+,\\s*Loader=yaml\\.Loader\\)"
        message: "Insecure deserialization detected with yaml.load(). Use yaml.safe_load() instead for untrusted input."
        
      # Pattern 3: Insecure deserialization with marshal
      - pattern: "marshal\\.loads\\(|marshal\\.load\\("
        message: "Insecure deserialization detected with marshal. Marshal is not secure against maliciously constructed data."
        
      # Pattern 4: Insecure deserialization with shelve
      - pattern: "shelve\\.open\\("
        message: "Potentially insecure deserialization with shelve detected. Shelve uses pickle internally and is not secure against malicious data."
        
      # Pattern 5: Insecure use of eval or exec
      - pattern: "eval\\(|exec\\(|compile\\([^,]+,\\s*['\"][^'\"]+['\"]\\s*,\\s*['\"]exec['\"]\\)"
        message: "Insecure use of eval() or exec() detected. These functions can execute arbitrary code and should never be used with untrusted input."
        
      # Pattern 6: Missing integrity verification for downloads
      - pattern: "urllib\\.request\\.urlretrieve\\(|requests\\.get\\([^)]*\\.exe['\"]\\)|requests\\.get\\([^)]*\\.zip['\"]\\)|requests\\.get\\([^)]*\\.tar\\.gz['\"]\\)"
        message: "File download without integrity verification detected. Always verify the integrity of downloaded files using checksums or digital signatures."
        
      # Pattern 7: Insecure package installation
      - pattern: "pip\\s+install\\s+[^-]|subprocess\\.(?:call|run|Popen)\\(['\"]pip\\s+install"
        message: "Insecure package installation detected. Specify package versions and consider using hash verification for pip installations."
        
      # Pattern 8: Missing integrity checks for configuration
      - pattern: "config\\.read\\(|json\\.loads?\\(|yaml\\.safe_load\\(|toml\\.loads?\\("
        message: "Configuration loading detected. Ensure integrity verification for configuration files, especially in production environments."
        
      # Pattern 9: Insecure temporary file creation
      - pattern: "tempfile\\.mktemp\\(|os\\.tempnam\\(|os\\.tmpnam\\("
        message: "Insecure temporary file creation detected. Use tempfile.mkstemp() or tempfile.TemporaryFile() instead to avoid race conditions."
        
      # Pattern 10: Insecure file operations with untrusted paths
      - pattern: "open\\([^,)]+\\+\\s*request\\.|open\\([^,)]+\\+\\s*user_|open\\([^,)]+\\+\\s*input\\("
        message: "Potentially insecure file operation with user-controlled path detected. Validate and sanitize file paths from untrusted sources."
        
      # Pattern 11: Missing integrity checks for updates
      - pattern: "auto_update|self_update|check_for_updates"
        message: "Update mechanism detected. Ensure proper integrity verification for software updates using digital signatures or secure checksums."
        
      # Pattern 12: Insecure plugin or extension loading
      - pattern: "importlib\\.import_module\\(|__import__\\(|load_plugin|load_extension|load_module"
        message: "Dynamic module loading detected. Implement integrity checks and validation before loading external modules or plugins."
        
      # Pattern 13: Insecure use of subprocess with shell=True
      - pattern: "subprocess\\.(?:call|run|Popen)\\([^,)]*shell\\s*=\\s*True"
        message: "Insecure subprocess execution with shell=True detected. This can lead to command injection if user input is involved."
        
      # Pattern 14: Missing integrity verification for serialized data
      - pattern: "json\\.loads?\\([^,)]*request\\.|json\\.loads?\\([^,)]*user_|json\\.loads?\\([^,)]*input\\("
        message: "Deserialization of user-controlled data detected. Implement schema validation or integrity checks before processing."
        
      # Pattern 15: Insecure use of globals or locals
      - pattern: "globals\\(\\)\\[|locals\\(\\)\\["
        message: "Potentially insecure modification of globals or locals detected. This can lead to unexpected behavior or security issues."

  - type: suggest
    message: |
      **Python Software and Data Integrity Best Practices:**
      
      1. **Secure Deserialization:**
         - Avoid using pickle, marshal, or shelve with untrusted data

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ivangrynenko/cursorrules](https://github.com/ivangrynenko/cursorrules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
