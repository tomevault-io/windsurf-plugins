---
trigger: always_on
description: Detect and prevent cryptographic failures in JavaScript applications as defined in OWASP Top 10:2021-A02
---

# JavaScript Cryptographic Failures (OWASP A02:2021)

<rule>
name: javascript_cryptographic_failures
description: Detect and prevent cryptographic failures in JavaScript applications as defined in OWASP Top 10:2021-A02

actions:
  - type: enforce
    conditions:
      # Pattern 1: Weak or insecure cryptographic algorithms
      - pattern: "(?:createHash|crypto\\.createHash)\\(['\"](?:md5|sha1)['\"]\\)|(?:crypto|require\\(['\"]crypto['\"]\\))\\.(?:createHash|Hash)\\(['\"](?:md5|sha1)['\"]\\)|new (?:MD5|SHA1)\\(|CryptoJS\\.(?:MD5|SHA1)\\("
        message: "Using weak hashing algorithms (MD5/SHA1). Use SHA-256 or stronger algorithms."
        
      # Pattern 2: Hardcoded secrets/credentials
      - pattern: "(?:const|let|var)\\s+(?:password|secret|key|token|auth|apiKey|api_key)\\s*=\\s*['\"][^'\"]+['\"]"
        message: "Potential hardcoded credentials detected. Store secrets in environment variables or a secure vault."
        
      # Pattern 3: Insecure random number generation
      - pattern: "Math\\.random\\(\\)|Math\\.floor\\(\\s*Math\\.random\\(\\)\\s*\\*"
        message: "Using Math.random() for security purposes. Use crypto.randomBytes() or Web Crypto API for cryptographic operations."
        
      # Pattern 4: Weak SSL/TLS configuration
      - pattern: "(?:tls|https|require\\(['\"]https['\"]\\)|require\\(['\"]tls['\"]\\))\\.(?:createServer|request|get)\\([^\\)]*?{[^}]*?secureProtocol\\s*:\\s*['\"](?:SSLv2_method|SSLv3_method|TLSv1_method|TLSv1_1_method)['\"]"
        message: "Using deprecated/insecure SSL/TLS protocol versions. Use TLS 1.2+ for secure communications."
        
      # Pattern 5: Missing certificate validation
      - pattern: "(?:rejectUnauthorized|strictSSL)\\s*:\\s*false"
        message: "SSL certificate validation is disabled. Always validate certificates in production environments."
        
      # Pattern 6: Insecure cipher usage
      - pattern: "(?:createCipheriv|crypto\\.createCipheriv)\\(['\"](?:des|des3|rc4|bf|blowfish|aes-\\d+-ecb)['\"]"
        message: "Using insecure encryption cipher or mode. Use AES with GCM or CBC mode with proper padding."
        
      # Pattern 7: Insufficient key length
      - pattern: "(?:generateKeyPair|generateKeyPairSync)\\([^,]*?['\"]rsa['\"][^,]*?{[^}]*?modulusLength\\s*:\\s*(\\d{1,3}|1[0-9]{3}|20[0-3][0-9]|204[0-7])\\s*}"
        message: "Using insufficient key length for asymmetric encryption. RSA keys should be at least 2048 bits, preferably 4096 bits."
        
      # Pattern 8: Insecure password hashing
      - pattern: "(?:createHash|crypto\\.createHash)\\([^)]*?\\)\\.(?:update|digest)\\([^)]*?\\)|CryptoJS\\.(?:SHA256|SHA512|SHA3)\\([^)]*?\\)"
        negative_pattern: "(?:bcrypt|scrypt|pbkdf2|argon2)"
        message: "Using plain hashing for passwords. Use dedicated password hashing functions like bcrypt, scrypt, or PBKDF2."
        
      # Pattern 9: Missing salt in password hashing
      - pattern: "(?:pbkdf2|pbkdf2Sync)\\([^,]+,[^,]+,[^,]+,\\s*\\d+\\s*,[^,]+\\)"
        negative_pattern: "(?:salt|crypto\\.randomBytes)"
        message: "Ensure you're using a proper random salt with password hashing functions."
        
      # Pattern 10: Insecure cookie settings
      - pattern: "(?:document\\.cookie|cookies\\.set|res\\.cookie|cookie\\.serialize)\\([^)]*?\\)"
        negative_pattern: "(?:secure\\s*:|httpOnly\\s*:|sameSite\\s*:)"
        message: "Cookies with sensitive data should have secure and httpOnly flags enabled."
        
      # Pattern 11: Client-side encryption
      - pattern: "(?:encrypt|decrypt|createCipher|createDecipher)\\([^)]*?\\)"
        location: "(?:frontend|client|browser|react|vue|angular)"
        message: "Performing sensitive cryptographic operations on the client side. Move encryption/decryption logic to the server."
        
      # Pattern 12: Insecure JWT implementation
      - pattern: "(?:jwt\\.sign|jsonwebtoken\\.sign)\\([^,]*?,[^,]*?,[^\\)]*?\\)"
        negative_pattern: "(?:expiresIn|algorithm\\s*:\\s*['\"](?:HS256|HS384|HS512|RS256|RS384|RS512|ES256|ES384|ES512)['\"])"
        message: "JWT implementation missing expiration or using weak algorithm. Set expiresIn and use a strong algorithm."
        
      # Pattern 13: Weak PRNG in Node.js
      - pattern: "(?:crypto\\.pseudoRandomBytes|crypto\\.rng|crypto\\.randomInt)\\("
        message: "Using potentially weak pseudorandom number generator. Use crypto.randomBytes() for cryptographic security."
        
      # Pattern 14: Insecure local storage usage for sensitive data
      - pattern: "(?:localStorage\\.setItem|sessionStorage\\.setItem)\\(['\"](?:token|auth|jwt|password|secret|key|credential)['\"]"
        message: "Storing sensitive data in browser storage. Use secure HttpOnly cookies for authentication tokens."
        
      # Pattern 15: Weak password validation
      - pattern: "(?:password\\.length\\s*>=?\\s*\\d|password\\.match\\(['\"][^'\"]+['\"]\\))"
        negative_pattern: "(?:password\\.length\\s*>=?\\s*(?:8|9|10|11|12)|[A-Z]|[a-z]|[0-9]|[^A-Za-z0-9])"
        message: "Weak password validation. Require at least 12 characters with a mix of uppercase, lowercase, numbers, and special characters."

  - type: suggest
    message: |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ivangrynenko/cursorrules](https://github.com/ivangrynenko/cursorrules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
