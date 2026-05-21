---
trigger: always_on
description: Detect and prevent cryptographic failures in Python applications as defined in OWASP Top 10:2021-A02
---

# Python Cryptographic Failures Security Standards (OWASP A02:2021)

This rule enforces security best practices to prevent cryptographic failures in Python applications, as defined in OWASP Top 10:2021-A02.

<rule>
name: python_cryptographic_failures
description: Detect and prevent cryptographic failures in Python applications as defined in OWASP Top 10:2021-A02
filters:
  - type: file_extension
    pattern: "\\.(py)$"
  - type: file_path
    pattern: ".*"

actions:
  - type: enforce
    conditions:
      # Pattern 1: Weak or insecure cryptographic algorithms
      - pattern: "import\\s+(md5|sha1)|hashlib\\.(md5|sha1)\\(|Crypto\\.Hash\\.(MD5|SHA1)|cryptography\\.hazmat\\.primitives\\.hashes\\.(MD5|SHA1)"
        message: "Using weak hashing algorithms (MD5/SHA1). Use SHA-256 or stronger algorithms from the hashlib or cryptography packages."
        
      # Pattern 2: Hardcoded secrets/credentials
      - pattern: "(password|secret|key|token|auth)\\s*=\\s*['\"][^'\"]+['\"]"
        message: "Potential hardcoded credentials detected. Store secrets in environment variables or a secure vault."
        
      # Pattern 3: Insecure random number generation
      - pattern: "random\\.(random|randint|choice|sample)|import random"
        message: "Using Python's standard random module for security purposes. Use secrets module or cryptography.hazmat.primitives.asymmetric for cryptographic operations."
        
      # Pattern 4: Weak SSL/TLS configuration
      - pattern: "ssl\\.PROTOCOL_(SSLv2|SSLv3|TLSv1|TLSv1_1)|SSLContext\\(\\s*ssl\\.PROTOCOL_(SSLv2|SSLv3|TLSv1|TLSv1_1)\\)"
        message: "Using deprecated/insecure SSL/TLS protocol versions. Use TLS 1.2+ (ssl.PROTOCOL_TLS_CLIENT with minimum version set)."
        
      # Pattern 5: Missing certificate validation
      - pattern: "verify\\s*=\\s*False|check_hostname\\s*=\\s*False|CERT_NONE"
        message: "SSL certificate validation is disabled. Always validate certificates in production environments."
        
      # Pattern 6: Insecure cipher usage
      - pattern: "DES|RC4|Blowfish|ECB"
        message: "Using insecure encryption cipher or mode. Use AES with GCM or CBC mode with proper padding."
        
      # Pattern 7: Insufficient key length
      - pattern: "RSA\\([^,]+,\\s*[0-9]+\\s*\\)|key_size\\s*=\\s*([0-9]|10[0-9][0-9]|11[0-9][0-9]|12[0-4][0-9])"
        message: "Using insufficient key length for asymmetric encryption. RSA keys should be at least 2048 bits, preferably 4096 bits."
        
      # Pattern 8: Insecure password hashing
      - pattern: "\\.encode\\(['\"]utf-?8['\"]\\)\\.(digest|hexdigest)\\(\\)|hashlib\\.[a-zA-Z0-9]+\\([^)]*\\)\\.(digest|hexdigest)\\(\\)"
        message: "Using plain hashing for passwords. Use dedicated password hashing functions like bcrypt, Argon2, or PBKDF2."
        
      # Pattern 9: Missing salt in password hashing
      - pattern: "pbkdf2_hmac\\([^,]+,[^,]+,[^,]+,\\s*[0-9]+\\s*\\)"
        message: "Ensure you're using a proper random salt with password hashing functions."
        
      # Pattern 10: Insecure cookie settings
      - pattern: "set_cookie\\([^)]*secure\\s*=\\s*False|set_cookie\\([^)]*httponly\\s*=\\s*False"
        message: "Cookies with sensitive data should have secure and httponly flags enabled."

  - type: suggest
    message: |
      **Python Cryptography Best Practices:**
      
      1. **Secure Password Storage:**
         - Use dedicated password hashing algorithms:
           ```python
           import bcrypt
           hashed = bcrypt.hashpw(password.encode('utf-8'), bcrypt.gensalt(rounds=12))
           ```
         - Or use Argon2 (preferred) or PBKDF2 with sufficient iterations:
           ```python
           from argon2 import PasswordHasher
           ph = PasswordHasher()
           hash = ph.hash(password)
           ```
      
      2. **Secure Random Number Generation:**
         - Use the `secrets` module for cryptographic operations:
           ```python
           import secrets
           token = secrets.token_hex(32)  # 256 bits of randomness
           ```
         - For cryptographic keys, use proper key generation functions:
           ```python
           from cryptography.hazmat.primitives.asymmetric import rsa
           private_key = rsa.generate_private_key(public_exponent=65537, key_size=4096)
           ```
      
      3. **Secure Communications:**
         - Use TLS 1.2+ for all communications:
           ```python
           import ssl
           context = ssl.create_default_context()
           context.minimum_version = ssl.TLSVersion.TLSv1_2
           ```
         - Always validate certificates:
           ```python
           import requests
           response = requests.get('https://example.com', verify=True)
           ```
      
      4. **Proper Key Management:**
         - Never hardcode secrets in source code
         - Use environment variables or secure vaults:
           ```python
           import os
           api_key = os.environ.get('API_KEY')
           ```
         - Consider using dedicated key management services
      
      5. **Secure Encryption:**
         - Use high-level libraries like `cryptography`:
           ```python
           from cryptography.fernet import Fernet

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ivangrynenko/cursorrules](https://github.com/ivangrynenko/cursorrules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
