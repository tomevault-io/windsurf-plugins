---
trigger: always_on
description: Token-saving Java file reader. Use when working with Java files (.java) to reduce token usage. Auto-triggers when exploring, reading, or understanding Java classes, services, controllers, entities, or any .java files. Run jskim before reading raw Java source when you need structural context first, then inspect only the lines you need.
---


# jskim — Java Token Saver for Spring Boot

A CLI tool that summarizes Java files compactly, saving 70-80% of input tokens. Optimized for Spring Boot projects with Lombok, REST controllers, DI wiring, and configuration properties.

## Requirements

Python 3.10+ — install via pip:
```bash
pip install jskim
```

**Before first use**, verify jskim is installed by running `jskim --version`. If you get "command not found", tell the user:
> jskim is not installed. Install it with: `pip install jskim`

Do not attempt to run jskim commands until it is confirmed installed. Fall back to your normal file-reading tools if the user declines to install it.

## Usage

`jskim` auto-detects whether you're pointing at a file or directory, and whether you're asking for a summary or method extraction.

### Single file summary
Summarizes a Java file — collapses imports, fields, boilerplate (getters/setters/equals/hashCode), and shows method signatures with line ranges. Shows annotation parameters for key Spring annotations (`@GetMapping("/path")`, `@Value("${key}")`, `@ConfigurationProperties("prefix")`, etc.).

```bash
jskim <file.java>
jskim <file.java> --grep <pattern>       # filter methods by name/signature
jskim <file.java> --annotation <@Ann>    # filter methods by annotation
jskim A.java B.java C.java               # multiple files
```

Java simple source files without an explicit type wrapper are summarized as `implicit class <FileStem>`, and their top-level methods are treated like normal class methods.

**Filters** (useful for large files with many methods):
- `--grep billing` — show only methods whose signature contains "billing" (case-insensitive)
- `--annotation @Transactional` — show only methods with that annotation
- Filters apply to the method listing only. Header, fields, and inner types are always shown.
- Filters can be combined: `--grep create --annotation @PostMapping`

### Project map
Generates a compact map of all Java files in a directory — packages, classes, annotations, field/method counts, Lombok usage, enum constants.

```bash
jskim <src_dir>
jskim <src_dir> --deps                          # import-based dependencies
jskim <src_dir> --endpoints                     # REST endpoint map
jskim <src_dir> --beans                         # Spring bean DI graph + @Bean producers + config properties
jskim <src_dir> --callers Class.method          # upstream callers for a specific method
jskim <src_dir> --impact Class.method           # callers + direct callees for a specific method
jskim <src_dir> --impact Class.method --depth 2 # bounded caller/callee hierarchy depth
jskim <src_dir> --package <prefix>               # filter by package
jskim <src_dir> --annotation <@Ann>              # filter by class annotation
jskim <src_dir> --extends <ClassName>            # filter by superclass
jskim <src_dir> --implements <Interface>        # filter by implemented interface
```

**Filters** (essential for large projects with hundreds of files):
- `--package com.stw.server.tripsheet` — only show classes in that package (prefix match)
- `--annotation @RestController` — only show classes with that annotation
- `--extends BaseService` — only show classes extending that superclass
- `--implements EventPublisher` — only show classes implementing that interface
- `--deps` — show which classes depend on which (uses imports, runs in seconds even on 2000+ files)
- `--endpoints` — list all REST endpoints: HTTP method, path, handler method, line number
- `--beans` — show Spring bean DI graph, `@Bean` factory method producers, and `@ConfigurationProperties` with field details
- `--callers BillingService.create` — show resolved upstream callers for a class-qualified method target
- `--impact BillingService.create` — show callers plus resolved downstream calls from the target method
- `--depth 2` — follow caller/callee edges beyond direct neighbors; default is 1 and usually best
- Filters can be combined: `--package com.example --annotation @Service --deps --endpoints --beans`

**Call hierarchy rules:**
- Always use a class-qualified target (`Class.method` or `com.example.Class.method`). Bare method names like `create` are intentionally rejected because they are too ambiguous in Java projects.
- If simple class names collide, rerun with the fully-qualified class name shown in the candidates list.
- Edges are resolved from same-class calls and field calls where the field type is a project class, such as `billingService.create()`.
- Calls on local variables, parameters, or overloaded targets may be skipped when they cannot be resolved safely. Treat missing edges as "not proven" rather than "not called."

### Diff mode
Summarizes only the Java files and methods changed in a git diff. Ideal for PR reviews — instead of reading full files, get structural context for just the changed parts.

```bash
jskim --diff HEAD~1                    # changes since last commit
jskim --diff main                      # changes vs main branch

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [garvit-joshi/jskim](https://github.com/garvit-joshi/jskim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
