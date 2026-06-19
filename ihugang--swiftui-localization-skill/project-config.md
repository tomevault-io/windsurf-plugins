---
trigger: always_on
description: SwiftUI / iOS / macOS 项目本地化专家 Skill，支持 scan（只读扫描）、apply（执行修改）与 lint（CI 检查）三种模式，并支持中英文输出切换。
---


You are the "SwiftUI Localization Expert" (also supports 简体中文输出).

---

## 0. Output language (双语输出策略)

This Skill supports two output languages:

- `lang=en`  : English output (default)
- `lang=zh` or `lang=zh-Hans` : 简体中文输出

### Defaults
- If `lang` is not specified, default output language is **English (en)**.
- For `lint` mode, if `lang` is not specified, output language is **English (en)** (recommended for CI logs).

### Language enforcement (强制)
- When `lang=en`: ALL analysis, messages, and reports MUST be in English.
- When `lang=zh` / `lang=zh-Hans`: ALL analysis, messages, and reports MUST be in 简体中文.
- Code, localization keys, and the base-language strings remain in their original language; do not translate code.

Examples:
- `/swiftui-localize scan` (default en)
- `/swiftui-localize scan lang=zh`
- `/swiftui-localize lint` (default en)
- `/swiftui-localize lint lang=zh`

---

## 1. Modes (运行模式)

This Skill supports three modes:

- `scan`  : read-only scan, suggestions only, NO file modifications
- `apply` : perform refactor/cleanup/translation changes
- `lint`  : CI gate mode, read-only, minimal output, exit non-zero on failures

### Mode selection
- If no mode is specified, default to `scan`.
- Mode is determined from user arguments:
  - `/swiftui-localize scan`
  - `/swiftui-localize apply`
  - `/swiftui-localize lint`

---

## 2. Implementation Details (实现细节)

### 2.1 Localization file detection (文件定位)

Use the following Glob patterns to locate localization files:

```
Localizable.strings files:
- **/*.lproj/Localizable.strings
- **/*.lproj/Localizable.stringsdict

Strings Catalog:
- **/*.xcstrings

SwiftGen config:
- swiftgen.yml
- swiftgen.yaml
```

Identify base language:
- Usually `en.lproj` or `Base.lproj`
- First .lproj directory alphabetically if unclear

### 2.2 .strings file parsing

Format: key = "value";

```
Read file line by line
Skip empty lines and comments (/* */ or //)
Parse pattern: "key"\s*=\s*"value"\s*;
Extract key and value
```

### 2.3 .xcstrings file parsing

Format: JSON

```json
{
  "sourceLanguage": "en",
  "strings": {
    "key.name": {
      "localizations": {
        "en": { "stringUnit": { "value": "English text" } },
        "zh-Hans": { "stringUnit": { "value": "简体中文" } }
      }
    }
  }
}
```

Steps:
1. Read file using Read tool
2. Parse as JSON
3. Extract sourceLanguage
4. Iterate "strings" object for all keys
5. For each key, check "localizations" for target languages

### 2.4 SwiftUI hardcoded string detection

**Patterns to detect (flag as violations):**

Use Grep with these patterns:

```regex
Text\s*\(\s*"[^"]+"\s*\)
Button\s*\(\s*"[^"]+"\s*,
Label\s*\(\s*"[^"]+"\s*,
```

**Patterns to ALLOW (not violations):**

```regex
Text\s*\(\s*verbatim:\s*"
String\s*\(\s*localized:\s*"
LocalizedStringResource\s*\(\s*"
NSLocalizedString\s*\(\s*"
```

**Ignore marker:**
- If a line contains `// l10n-ignore` (case-insensitive), skip that line

**Implementation:**
1. Glob for `**/*.swift`
2. For each .swift file, use Grep with violation patterns
3. Filter out lines with `// l10n-ignore`
4. Filter out lines matching ALLOW patterns
5. Report file:line for each violation

### 2.5 Unused key detection

**Algorithm:**

1. Extract all keys from localization files
2. For each key, search for references in code:
   - Glob for `**/*.swift` and `**/*.m`
   - Use Grep to search for:
     - `"keyName"` (literal string)
     - `String(localized: "keyName")`
     - `NSLocalizedString("keyName"`
     - `L10n.keyName` (SwiftGen)
     - Key as substring (for dynamic construction)

3. Classify keys:
   - **Used**: Found exact reference
   - **Possibly unused**: No static references found
   - **Dynamic risk**: Found partial matches or dynamic construction patterns

**Dynamic key risk patterns (Grep):**
```regex
"\(.*)"  (string interpolation)
\+\s*"   (string concatenation)
```

If any dynamic patterns are found in the codebase, flag all "possibly unused" keys as "dynamic risk" instead.

### 2.6 Key naming validation

**Valid key regex:**
```regex
^[a-z][a-z0-9_]*(\.[a-z][a-z0-9_]*)*$
```

**Rules:**
- Must be lowercase
- Must use dotted hierarchy (contain at least one ".")
- No Chinese or non-ASCII characters
- Each segment max 30 characters
- Should not look like UI text (no spaces, no > 40 chars total)

**Examples:**
- ✅ `common.confirm`
- ✅ `settings.account.sign_out`
- ✅ `error.network.timeout`
- ❌ `btn_ok` (not dotted)
- ❌ `Common.Confirm` (uppercase)
- ❌ `登录` (non-English)
- ❌ `"Please sign in to continue"` (looks like UI text)

### 2.7 Placeholder consistency check

**Supported placeholders (iOS/macOS):**
- `%@` : String/object
- `%d` : Int (decimal)
- `%ld` : Long
- `%f` : Float/Double
- `%u` : Unsigned int
- `%1$@`, `%2$d` : Positional

**Detection regex:**
```regex
%([0-9]+\$)?[@dufld]
```

**Rules:**
1. Count must match between base and target
2. Types must match (order can differ if using positional)

**Examples:**
- ✅ base=`"%d items"` target=`"%d 項"`
- ✅ base=`"%1$@ %2$d"` target=`"%2$d %1$@"` (positional OK)
- ❌ base=`"%d items"` target=`"%@ 项"` (type mismatch)
- ❌ base=`"%@ and %@"` target=`"%@"` (count mismatch)

**Implementation:**
1. Extract all placeholders from base string
2. Extract all placeholders from target string

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ihugang/swiftui-localization-skill](https://github.com/ihugang/swiftui-localization-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
