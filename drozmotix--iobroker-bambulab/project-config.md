---
trigger: always_on
description: ioBroker.bambulab is a Node.js adapter for integrating Bambulab 3D printers into the ioBroker home automation platform. The adapter connects to Bambulab printers via MQTT to retrieve print data and control main printer functions.
---

# ioBroker.bambulab Development Instructions

ioBroker.bambulab is a Node.js adapter for integrating Bambulab 3D printers into the ioBroker home automation platform. The adapter connects to Bambulab printers via MQTT to retrieve print data and control main printer functions.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Prerequisites and Setup
- Node.js version 20 or higher (engines requirement in package.json)
- npm package manager (comes with Node.js)

### Bootstrap, Build, and Test the Repository
1. **Install dependencies**: 
   - `npm install` -- takes 26 seconds. NEVER CANCEL. Set timeout to 60+ minutes for safety.
   - Produces deprecation warnings but these can be ignored - the installation succeeds.
   - May report vulnerabilities - these are in dependencies and do not affect functionality.

2. **Lint the code**:
   - `npm run lint` -- takes less than 1 second. Uses ESLint with custom rules.
   - Must pass before committing changes or CI will fail.

3. **Run tests**:
   - `npm run test` -- takes 1 second. NEVER CANCEL. Set timeout to 30+ minutes for safety.
   - Runs both unit tests (`npm run test:js`) and package validation (`npm run test:package`)
   - All tests must pass before committing.

4. **Run integration tests**:
   - `npm run test:integration` -- takes 44 seconds. NEVER CANCEL. Set timeout to 60+ minutes.
   - Creates temporary test environment with Redis and ioBroker infrastructure.
   - Expects network access to Bambulab servers (will show connection errors but adapter should start successfully).

### Translation Management

**CRITICAL**: Translation files must stay synchronized with `admin/jsonConfig.json`. Orphaned keys or missing translations will cause UI issues and PR review delays.

#### Overview
- **Location**: `admin/i18n/{lang}/translations.json` for 11 languages (de, en, es, fr, it, nl, pl, pt, ru, uk, zh-cn)
- **Source of truth**: `admin/jsonConfig.json` - all `label` and `help` properties must have translations
- **Command**: `npm run translate` - auto-generates translations but does NOT remove orphaned keys
- **Formatting**: English uses tabs, other languages use 4 spaces (per ioBroker standards)

#### Critical Rules
1. **Keys must match exactly**: Every label/help text in `jsonConfig.json` must exist in all translation files
2. **No orphaned keys**: Translation files must NOT contain keys that don't exist in `jsonConfig.json`
3. **Native language required**: All translations must be in the target language (no English fallbacks)
4. **Alphabetical sorting**: Keys must be sorted alphabetically to prevent merge conflicts

#### Workflow for Translation Updates

**When modifying `admin/jsonConfig.json`:**

1. **Make your changes** to labels, help texts, or add new UI elements
2. **Run automatic translation**:
   ```bash
   npm run translate
   ```
   This updates io-package.json and adds new keys to translation files, but **does NOT remove orphaned keys**

3. **Run validation script** to check for issues:
   ```javascript
   // Create validation script
   const fs = require('fs');
   const jsonConfig = JSON.parse(fs.readFileSync('admin/jsonConfig.json', 'utf8'));
   
   function extractTexts(obj, texts = new Set()) {
       if (typeof obj === 'object' && obj !== null) {
           if (obj.label) texts.add(obj.label);
           if (obj.help) texts.add(obj.help);
           for (const key in obj) {
               extractTexts(obj[key], texts);
           }
       }
       return texts;
   }
   
   const requiredTexts = extractTexts(jsonConfig);
   const languages = ['de', 'en', 'es', 'fr', 'it', 'nl', 'pl', 'pt', 'ru', 'uk', 'zh-cn'];
   
   languages.forEach(lang => {
       const translations = JSON.parse(fs.readFileSync(`admin/i18n/${lang}/translations.json`, 'utf8'));
       const translationKeys = new Set(Object.keys(translations));
       
       const missing = Array.from(requiredTexts).filter(text => !translationKeys.has(text));
       const orphaned = Array.from(translationKeys).filter(key => !requiredTexts.has(key));
       
       console.log(`\n=== ${lang} ===`);
       if (missing.length > 0) {
           console.log('❌ Missing keys:', missing);
       }
       if (orphaned.length > 0) {
           console.log('❌ Orphaned keys (REMOVE THESE):', orphaned);
       }
       if (missing.length === 0 && orphaned.length === 0) {
           console.log('✅ All keys match!');
       }
   });
   ```

4. **Remove orphaned keys manually** - if validation shows orphaned keys, remove them from all translation files
5. **Add missing translations** - if validation shows missing keys, add proper native language translations
6. **Verify all languages** - ensure all 11 languages pass validation
7. **Run mandatory validation**:
   ```bash
   npm run lint    # Must pass
   npm run test    # Must pass
   ```

#### Common Pitfalls (Lessons from PR #202)

❌ **Don't do this:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DrozmotiX/ioBroker.bambulab](https://github.com/DrozmotiX/ioBroker.bambulab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
