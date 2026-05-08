---
trigger: always_on
description: **See also:** [REQUIREMENTS.md](REQUIREMENTS.md) for behavioral requirements (rate limiting, 429 handling, robots.txt compliance, etc.)
---

# DuckDB Crawler Extension - Developer Guide

**See also:** [REQUIREMENTS.md](REQUIREMENTS.md) for behavioral requirements (rate limiting, 429 handling, robots.txt compliance, etc.)

## Code Organization Guidelines

### File Size Limits
- **Max ~500 lines per file** - if approaching this, plan to split
- **Max ~100 lines per function** - extract helpers if longer
- Files over 1000 lines MUST be refactored before adding more code

### When to Refactor
Before adding new features to a large file, split it first:
1. Identify cohesive groups (e.g., thread utils, HTTP handling, parsing)
2. Extract to new `.cpp/.hpp` files
3. Update `CMakeLists.txt` to include new sources
4. Then add the new feature

### Suggested Structure
```
src/
├── crawler_extension.cpp  # Entry point, registration only
├── crawler_function.cpp   # Table function binding, main orchestration
├── crawler_worker.cpp     # CrawlWorker thread logic
├── thread_utils.cpp       # ThreadSafeQueue, ThreadSafeDomainMap
├── sitemap_discovery.cpp  # Sitemap fetching/caching
├── utils.cpp              # DecompressGzip, GenerateSurtKey, helpers
├── http_client.cpp        # HTTP via libcurl
├── robots_parser.cpp      # robots.txt parsing
├── sitemap_parser.cpp     # XML sitemap parsing
├── link_parser.cpp        # HTML link extraction
└── include/               # Headers
```

## Build Commands

```bash
# First time setup
git clone https://github.com/microsoft/vcpkg.git
./vcpkg/bootstrap-vcpkg.sh

# Build with ninja (faster)
make release GEN=ninja VCPKG_TOOLCHAIN_PATH=$(pwd)/vcpkg/scripts/buildsystems/vcpkg.cmake

# Clean build
rm -rf build && make release GEN=ninja VCPKG_TOOLCHAIN_PATH=$(pwd)/vcpkg/scripts/buildsystems/vcpkg.cmake

# Run tests
./build/release/test/unittest --test-dir test/sql '*crawl*'

# Test manually
./build/release/duckdb -unsigned -c "
LOAD 'build/release/extension/crawler/crawler.duckdb_extension';
CRAWL (SELECT 'https://example.com/') INTO pages WITH (max_crawl_pages 5);
SELECT * FROM pages;
"
```

## Adding vcpkg Packages

Edit `vcpkg.json`:

```json
{
    "dependencies": [
        "zlib",
        {
            "name": "curl",
            "default-features": false,
            "features": ["ssl", "openssl", "http2"]
        },
        "new-package-name"
    ]
}
```

Platform-specific deps:
```json
{
    "name": "some-lib",
    "platform": "!windows"
}
```

Then update `CMakeLists.txt`:
```cmake
find_package(NewPackage REQUIRED)
target_link_libraries(${EXTENSION_NAME} NewPackage::lib)
target_link_libraries(${LOADABLE_EXTENSION_NAME} NewPackage::lib)
```

## Adding Custom Keywords (Parser Extension)

### 1. Register Parser Extension

In `src/crawler_extension.cpp`:
```cpp
static void LoadInternal(ExtensionLoader &loader) {
    auto &config = DBConfig::GetConfig(loader.GetDatabaseInstance());

    ParserExtension parser_ext;
    parser_ext.parse_function = MyParserExtension::Parse;
    parser_ext.plan_function = MyParserExtension::Plan;
    config.parser_extensions.push_back(std::move(parser_ext));
}
```

### 2. Implement Parser

In `src/crawl_parser.cpp`:

```cpp
// Check if statement starts with your keyword
ParserExtensionParseResult MyParserExtension::Parse(ParserExtensionInfo *info, const string &query) {
    string lower = StringUtil::Lower(query);

    // Return empty result if not our statement
    if (!StringUtil::StartsWith(lower, "mykeyword")) {
        return ParserExtensionParseResult();
    }

    // Parse the statement
    auto data = make_uniq<MyParseData>();

    // Find keywords: INTO, WHERE, WITH
    size_t into_pos = lower.find("into");
    size_t where_pos = lower.find("where");
    size_t with_pos = lower.find("with");

    // Extract table name after INTO
    if (into_pos != string::npos) {
        // Parse table name...
        data->table_name = extracted_name;
    }

    // Parse WITH options: WITH (key1 value1, key2 'value2')
    if (with_pos != string::npos) {
        ParseWithOptions(options_str, *data);
    }

    return ParserExtensionParseResult(make_uniq_base<ParserExtensionParseData, MyParseData>(std::move(data)));
}
```

### 3. Implement Planner

Convert parsed data to executable plan:

```cpp
ParserExtensionPlanResult MyParserExtension::Plan(ParserExtensionInfo *info,
                                                   ClientContext &context,
                                                   unique_ptr<ParserExtensionParseData> parse_data) {
    auto &data = dynamic_cast<MyParseData &>(*parse_data);

    // Build SQL to execute
    string sql = StringUtil::Format(
        "INSERT INTO %s SELECT * FROM my_table_function('%s')",
        data.table_name, data.some_param);

    // Parse and return the generated SQL
    Parser parser;
    parser.ParseQuery(sql);

    ParserExtensionPlanResult result;
    result.function = make_uniq<PlanExtensionFunction>();
    result.statements = std::move(parser.statements);
    return result;
}
```

## Predicate Pushdown Patterns

### URL Pattern Filtering (LIKE)

```cpp
// In crawler_function.cpp
static bool MatchesLikePattern(const std::string &url, const std::string &pattern) {
    // SQL LIKE: % = any chars, _ = single char
    size_t url_pos = 0, pat_pos = 0;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [midwork-finds-jobs/duckdb-crawler](https://github.com/midwork-finds-jobs/duckdb-crawler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
