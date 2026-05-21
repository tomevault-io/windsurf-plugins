---
trigger: always_on
description: Detect and prevent the use of vulnerable and outdated components in JavaScript applications as defined in OWASP Top 10:2021-A06
---

# JavaScript Vulnerable and Outdated Components (OWASP A06:2021)

<rule>
name: javascript_vulnerable_outdated_components
description: Detect and prevent the use of vulnerable and outdated components in JavaScript applications as defined in OWASP Top 10:2021-A06

actions:
  - type: enforce
    conditions:
      # Pattern 1: Outdated Package Versions in package.json
      - pattern: "\"(dependencies|devDependencies)\"\\s*:\\s*\\{[^}]*?\"([^\"]+)\"\\s*:\\s*\"\\^?([0-9]+\\.[0-9]+\\.[0-9]+)\""
        location: "package\\.json$"
        message: "Check for outdated dependencies in package.json. Regularly update dependencies to avoid known vulnerabilities."
        
      # Pattern 2: Direct CDN Links Without Integrity Hashes
      - pattern: "<script\\s+src=['\"]https?://(?:cdn|unpkg|jsdelivr)[^'\"]*['\"][^>]*(?!integrity=)"
        location: "\\.(html|js|jsx|ts|tsx)$"
        message: "CDN resources without integrity hashes. Add integrity and crossorigin attributes to script tags loading external resources."
        
      # Pattern 3: Hardcoded Library Versions in HTML
      - pattern: "<script\\s+src=['\"][^'\"]*(?:jquery|bootstrap|react|vue|angular|lodash|moment)[@-][0-9]+\\.[0-9]+\\.[0-9]+[^'\"]*['\"]"
        location: "\\.html$"
        message: "Hardcoded library versions in HTML. Consider using a package manager to manage dependencies."
        
      # Pattern 4: Deprecated Node.js APIs
      - pattern: "(?:new Buffer\\(|require\\(['\"]crypto['\"]\\)\\.createCipher\\(|require\\(['\"]crypto['\"]\\)\\.randomBytes\\([^,)]+\\)|require\\(['\"]fs['\"]\\)\\.exists\\()"
        message: "Using deprecated Node.js APIs. Replace with modern alternatives to avoid security and maintenance issues."
        
      # Pattern 5: Deprecated Browser APIs
      - pattern: "document\\.write\\(|document\\.execCommand\\(|escape\\(|unescape\\(|showModalDialog\\(|localStorage\\.clear\\(\\)|sessionStorage\\.clear\\(\\)"
        location: "(?:src|components|pages)"
        message: "Using deprecated browser APIs. Replace with modern alternatives to avoid compatibility and security issues."
        
      # Pattern 6: Insecure Dependency Loading
      - pattern: "require\\([^)]*?\\+\\s*[^)]+\\)|import\\([^)]*?\\+\\s*[^)]+\\)"
        message: "Dynamic dependency loading with variable concatenation. This can lead to dependency confusion attacks."
        
      # Pattern 7: Vulnerable Regular Expression Patterns (ReDoS)
      - pattern: "new RegExp\\([^)]*?(?:\\(.*\\)\\*|\\*\\+|\\+\\*|\\{\\d+,\\})"
        message: "Potentially vulnerable regular expression pattern that could lead to ReDoS attacks. Review and optimize the regex pattern."
        
      # Pattern 8: Insecure Package Installation
      - pattern: "npm\\s+install\\s+(?:--no-save|--no-audit|--no-fund|--force)"
        location: "(?:scripts|Dockerfile|docker-compose\\.yml|\\.github/workflows)"
        message: "Insecure package installation flags. Avoid using --no-audit, --no-save, or --force flags when installing packages."
        
      # Pattern 9: Missing Lock Files
      - pattern: "package\\.json"
        location: "package\\.json$"
        negative_pattern: "package-lock\\.json|yarn\\.lock|pnpm-lock\\.yaml"
        message: "Missing lock file. Use package-lock.json, yarn.lock, or pnpm-lock.yaml to ensure dependency consistency."
        
      # Pattern 10: Insecure Webpack Configuration
      - pattern: "webpack\\.config\\.js"
        location: "webpack\\.config\\.js$"
        negative_pattern: "(?:noEmitOnErrors|optimization\\.minimize)"
        message: "Potentially insecure webpack configuration. Consider enabling noEmitOnErrors and optimization.minimize."
        
      # Pattern 11: Outdated TypeScript Configuration
      - pattern: "\"compilerOptions\"\\s*:\\s*\\{[^}]*?\"target\"\\s*:\\s*\"ES5\""
        location: "tsconfig\\.json$"
        message: "Outdated TypeScript target. Consider using a more modern target like ES2020 for better security features."
        
      # Pattern 12: Insecure Package Sources
      - pattern: "registry\\s*=\\s*(?!https://registry\\.npmjs\\.org)"
        location: "\\.npmrc$"
        message: "Using a non-standard npm registry. Ensure you trust the source of your packages."
        
      # Pattern 13: Missing npm audit in CI/CD
      - pattern: "(?:ci|test|build)\\s*:\\s*\"[^\"]*?\""
        location: "package\\.json$"
        negative_pattern: "npm\\s+audit"
        message: "Missing npm audit in CI/CD scripts. Add 'npm audit' to your CI/CD pipeline to detect vulnerabilities."
        
      # Pattern 14: Insecure Import Maps
      - pattern: "<script\\s+type=['\"]importmap['\"][^>]*>[^<]*?\"imports\"\\s*:\\s*\\{[^}]*?\"[^\"]+\"\\s*:\\s*\"https?://[^\"]+\""
        negative_pattern: "integrity="
        message: "Insecure import maps without integrity checks. Add integrity hashes to import map entries."
        
      # Pattern 15: Outdated Polyfills
      - pattern: "(?:core-js|@babel/polyfill|es6-promise|whatwg-fetch)"
        message: "Using potentially outdated polyfills. Consider using modern alternatives or feature detection."

  - type: suggest
    message: |
      **JavaScript Vulnerable and Outdated Components Best Practices:**
      

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ivangrynenko/cursorrules](https://github.com/ivangrynenko/cursorrules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
