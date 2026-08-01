---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Common Commands

### Testing
- Run all tests: `./vendor/bin/phpunit`
- Run tests with coverage: `./vendor/bin/phpunit --coverage-clover=build/logs/clover.xml`
- Run specific test: `./vendor/bin/phpunit test/JiebaTest.php`
- Run custom POS tag tests: `./vendor/bin/phpunit test/CustomPosTagTest.php`
- Run security tests: `./vendor/bin/phpunit test/SecurityTest.php`
- Run user dictionary tests: `./vendor/bin/phpunit test/UserDictTest.php`
- Run memory management tests: `./vendor/bin/phpunit test/MemoryManagementTest.php`

### Code Quality
- Code style check: `./vendor/bin/phpcs`
- PSR2 standard check: `./vendor/bin/phpcs --standard=PSR2 --extensions=php --ignore="*/test/*" ./src/class`
- Code linting: `./vendor/bin/phpcs --standard=PSR2 src/`

### Demo Scripts
- Basic segmentation: `php src/cmd/demo.php`
- Keyword extraction: `php src/cmd/demo_extract_tags.php`
- Part-of-speech tagging: `php src/cmd/demo_posseg.php`
- Custom dictionary: `php src/cmd/demo_user_dict.php`
- Tokenization with positions: `php src/cmd/demo_tokenize.php`
- **Custom POS tagging**: `php src/cmd/demo_custom_pos_tag.php`
- **TF-IDF and POS integration**: `php src/cmd/demo_tf_idf_pos.php`
- **Mixed CJK language processing**: `php src/cmd/demo_mixed_cjk.php`

### Memory Requirements
All operations require significant memory allocation: `ini_set('memory_limit', '1024M');`

## Architecture Overview

This is a PHP port of the Python jieba Chinese text segmentation library. The core architecture consists of:

### Core Classes (src/class/)
- **Jieba**: Main segmentation engine with three modes (accurate, full, search)
  - Supports custom word addition with `addWord($word, $freq, $tag)`
  - Enhanced input validation and security measures
  - Memory management improvements
  - **NEW**: Support for `with_pos` and `with_scores` options in `cut()` method
- **Finalseg**: HMM-based final segmentation for unknown words using Viterbi algorithm
- **JiebaAnalyse**: TF-IDF keyword extraction functionality
  - **NEW**: Modular TF calculation with `calculateTF($words)`
  - **NEW**: Flexible TF-IDF calculation with `calculateTFIDF($tf_values, $detailed)`
- **Posseg**: Part-of-speech tagging with HMM model
  - **Custom POS tag support**: Add custom tags with `addWordTag($word, $tag)`
  - **Input validation**: Secure tag validation with length limits and character restrictions
  - **Memory cleanup**: `removeWordTag($word)` for tag cleanup
  - **NEW**: Support for `with_scores` option in `cut()` method
- **JiebaMemory**: NEW unified memory management utility
  - **Memory management**: `destroyAll()`, `initAll()`, `clearAllCaches()`
  - **Statistics**: `getMemoryStats()`, `getAllCacheStats()`, `getInitializationStatus()`
  - **Convenience**: `isAllInitialized()` for checking all classes

### Dictionary System (src/dict/)
- **dict.txt**: Default dictionary with word frequencies
- **dict.big.txt**: Traditional Chinese dictionary
- **dict.small.txt**: Compact dictionary for memory-constrained environments
- **user_dict.txt**: Custom user dictionary
- **idf.txt/idf.big.txt**: IDF values for keyword extraction
- **stop_words.txt**: Stop words for analysis
- **pos_tag_readable.txt**: Part-of-speech tag descriptions

### Model Files (src/model/)
- **prob_start.json**: HMM start probabilities
- **prob_trans.json**: HMM transition probabilities  
- **prob_emit.json**: HMM emission probabilities
- **pos/**: Part-of-speech specific HMM models

### Key Algorithms
1. **Trie-based DAG construction**: Efficient word graph scanning
2. **Dynamic programming**: Maximum probability path finding
3. **HMM + Viterbi**: Unknown word recognition
4. **TF-IDF**: Keyword extraction

### Initialization Pattern
All classes follow this pattern:
```php
Jieba::init($options);      // Load dictionary and build trie
Finalseg::init();           // Load HMM models
JiebaAnalyse::init();       // Load IDF data
Posseg::init();             // Load POS models

// NEW: Convenient initialization of all classes
JiebaMemory::initAll($options);  // Initialize all classes at once
```

### Dictionary Modes
- `'dict'=>'default'`: Standard dictionary
- `'dict'=>'big'`: Traditional Chinese support
- `'dict'=>'small'`: Memory-efficient mode
- `'dict'=>'test'`: Testing dictionary

### Memory Management
- Dictionary caching with .cache.json files
- Static class variables for model storage
- Large memory footprint requires 1GB+ allocation

### Multi-language Support
- Primary: Simplified/Traditional Chinese
- Secondary: Japanese, Korean (with `'cjk'=>'all'`)
- **ENHANCED**: Improved mixed-language text processing
- **NEW**: Better handling of complex mixed CJK scenarios
- Custom dictionaries can extend language support
- **NEW Demo**: `demo_mixed_cjk.php` for testing multi-language capabilities

## Enhanced TF-IDF and POS Integration Features

### NEW: Integrated TF-IDF Scoring
```php
// Jieba::cut() with POS tags
$pos_result = Jieba::cut($text, false, array('with_pos' => true));

// Jieba::cut() with TF-IDF scores
$scored_result = Jieba::cut($text, false, array('with_scores' => true));

// Jieba::cut() with both POS tags and TF-IDF scores

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [imaiwork/IMAI.WORK-AI-Phone](https://github.com/imaiwork/IMAI.WORK-AI-Phone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
