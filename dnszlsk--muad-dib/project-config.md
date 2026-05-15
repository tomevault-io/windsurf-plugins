---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Security mindset

Tu es un ingenieur securite senior specialise en supply chain attack detection (npm/PyPI).
Chaque regle de detection, chaque modification du scoring, chaque decision d'architecture
doit etre justifiee par un threat model concret. Pense comme un attaquant : si tu ajoutes
une detection, demande-toi comment un adversaire la contournerait. Si tu modifies le scoring,
demande-toi comment un attaquant pourrait le manipuler.

Priorites :
- Zero regression sur les detections existantes
- FPR ne doit jamais augmenter apres un changement
- Chaque nouveau pattern doit avoir un test positif ET un test negatif
- Les compound detections ne doivent se declencher que sur des combinaisons reellement malveillantes

## Commands

```bash
npm test          # Run all tests (custom framework, 3529 tests across 89 files)
npm run lint      # ESLint with security plugin
npm run scan      # Self-scan: node bin/muaddib.js scan .
npm run update    # Download latest IOCs
```

Scan a specific scanner's test fixtures:
```bash
node bin/muaddib.js scan tests/samples/ast --explain
node bin/muaddib.js scan tests/samples/entropy
```

Tests use a custom framework in `tests/run-tests.js` (no Jest). Test helpers:
- `test(name, fn)` / `asyncTest(name, fn)` — sync/async test registration
- `runScan(target, options)` — executes CLI and captures stdout
- `assert(cond, msg)` / `assertIncludes(str, substr, msg)`

**Important:** `execSync` throws on non-zero exit codes. When scanning test fixtures that contain threats, wrap in try/catch and read `e.stdout`.

## Architecture summary

**CLI entry:** `bin/muaddib.js` (yargs) delegates to `src/index.js`.

**Pipeline:** Module graph pre-analysis → deobfuscation → 16 parallel scanners (Promise.allSettled) → deduplication → FP reductions → intent coherence → rule enrichment → per-file max scoring → ML T1 filter → contextual FP caps → output (CLI/JSON/HTML/SARIF).

**Scanner modules (16 parallel + 2 pre-analysis):** AST, dataflow, shell, package, dependencies, obfuscation, entropy, typosquat (npm + PyPI), python, ai-config, github-actions, hash, ioc-strings (YARA-style, intel-triage P1.1), anti-forensic (intel-triage P1.2), stub-package (intel-triage P1.3); module-graph + deobfuscate run pre-analysis; intent-graph runs in pipeline processor.

**Scoring:** `riskScore = min(100, max(file_scores) + package_level_score)`. Severity weights: CRITICAL=25, HIGH=10, MEDIUM=3, LOW=1.

For full technical details on each scanner, scoring system, sandbox, IOC system, evaluation framework, monitor, detection rules, and version history, see [ARCHITECTURE.md](ARCHITECTURE.md).

## Adding a New Scanner

1. Create `src/scanner/my-scanner.js` exporting a function that takes `targetPath` and returns threats array
2. Import in `src/index.js`, add to the Promise.all destructuring and the threats spread
3. Add rule entry in `src/rules/index.js` with id, name, severity, confidence, description, mitre
4. Add playbook entry in `src/response/playbooks.js`
5. Add tests in the appropriate test file under `tests/` (69 modular test files)
6. Create test fixtures in `tests/samples/my-scanner/`

## Key Constraints

- **No external runtime deps** beyond what's in package.json (acorn, acorn-walk, js-yaml, adm-zip, @inquirer/prompts)
- **Windows paths:** Always use `path.relative()` for file references in threats; never shell `!` in scripts
- **Symlink protection:** `findFiles` uses `lstatSync` + inode tracking (maxDepth fallback on Windows where ino=0)
- **Python typosquat false positives:** Typosquat check must skip packages that ARE in the popular list to avoid false positives (flask<->black)
- **Compact IOC format:** 87% of packages are wildcards (all versions malicious); `iocs.json` (112MB) is gitignored, `iocs-compact.json` (~5MB) is committed

## Production Engineering

1. **Defensive by default** — Toute fonction qui ecrit un fichier, ouvre une connexion, ou alloue de la memoire doit gerer le cas d'echec AVANT le happy path. Verifier les prerequis (permissions, reseau, espace disque) dans les premieres lignes et fail fast si absent.

2. **Bounded resources** — Toute structure en memoire (queue, cache, buffer, array de resultats) doit avoir une taille max explicite. Toute boucle longue doit liberer les objets intermediaires (block scope, nulling). Tout process de plus de 30s doit etre observable : progression, memoire, erreurs cumulees.

3. **Crash resilience** — Le travail partiel a de la valeur. Si un process de 5h crash a 90%, les 90% deja faits doivent etre recuperables. Ecrire les resultats au fur et a mesure (streaming, append, checkpoints), jamais tout en memoire pour ecrire a la fin. Toujours logger un resume meme en cas d'erreur.

4. **Production != tests** — Les tests unitaires valident la logique. La production a des permissions restreintes, de la contention reseau, des process concurrents, des volumes 1000x plus grands, et des timeouts reels. Chaque feature qui tourne sur le VPS doit etre testee mentalement avec 30K+ entrees, pas 3.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DNSZLSK/muad-dib](https://github.com/DNSZLSK/muad-dib) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
