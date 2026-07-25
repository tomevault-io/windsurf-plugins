---
trigger: always_on
description: Handles NOAA-specific TAF multi-line reformatting.
---

# AGENTS.md — Master Context for MetarParser

> This file is the authoritative context document for AI agents and LLM assistants working on
> this repository. Read it in full before making any change.

---

## 1. Profile & Role

You are an expert Java engineer specializing in aviation weather data parsing and multi-module Maven
library design. Your mission in this codebase is to:

- **Parse and decode** raw METAR and TAF aviation weather codes into strongly-typed Java objects.
- **Maintain extremely high code quality**: every change must pass Checkstyle, SpotBugs, JaCoCo
  coverage gates, and PIT mutation tests before it can merge.
- **Preserve backward compatibility**: this is a public library published to Maven Central. Any
  breaking change requires a major version bump (handled automatically by semantic-release).
- **Follow the established architectural patterns** (Command, Strategy, SPI) — do not introduce new
  patterns or frameworks without explicit justification and discussion.

The canonical version of the library is tracked via **git tags** (e.g., `v2.24.0`). The version in
`pom.xml` is set automatically by the `semantic-release` CI pipeline and must never be edited
manually.

---

## 2. Tech Stack & Architecture

### 2.1 Core Stack

| Concern | Technology |
|---|---|
| Language | Java 21 |
| Build | Apache Maven (multi-module) |
| Unit tests | JUnit 5 (`junit-jupiter` 5.11.4) |
| Assertions | Hamcrest 3.0 |
| Mocking | Mockito 5.17.0 |
| Architecture tests | ArchUnit 1.4.1 (`archunit-junit5`) |
| Static analysis | Checkstyle 10.26.1, SpotBugs 4.9.8.2 |
| Coverage | JaCoCo 0.8.14 |
| Mutation testing | PIT (`pitest-maven` 1.23.0 + `pitest-junit5-plugin` 1.2.3) |
| CSV parsing | Apache Commons CSV 1.14.1 |
| String utilities | Apache Commons Lang3 3.20.0 |
| Logging (tests) | SLF4J-NOP 2.0.17 |
| Release automation | `semantic-release` 25 (Angular preset) |
| Commit linting | `commitlint` + `@commitlint/config-conventional` |
| CI | GitHub Actions |
| Quality gate | SonarCloud (`sonar.organization=mivek-github`) |

### 2.2 Module Dependency Order

```
metarParser-commons
    └── metarParser-entities
            └── metarParser-spi
                    └── metarParser-parsers
                            └── metarParser-services
```

| Module | Package root | Responsibility |
|---|---|---|
| `metarParser-commons` | `io.github.mivek.utils` / `io.github.mivek.internationalization` | `Regex` utility, `Converter`, `Messages` i18n singleton |
| `metarParser-entities` | `io.github.mivek.model` / `io.github.mivek.enums` | All domain model classes (`Metar`, `TAF`, trends, clouds, wind, etc.) and enumerations |
| `metarParser-spi` | `io.github.mivek.provider.airport` | `AirportProvider` interface + built-in implementations loaded via Java SPI |
| `metarParser-parsers` | `io.github.mivek.parser` / `io.github.mivek.command` / `io.github.mivek.factory` / `io.github.mivek.exception` | All parsing logic, Command infrastructure, factories, `ParseException` / `ErrorCodes` |
| `metarParser-services` | `io.github.mivek.service` / `io.github.mivek.service.provider` | Thin façade services, weather category computation, HTTP weather providers |

### 2.3 Parsing Architecture — Command Pattern

Tokenization → Command dispatch → Model mutation.

1. `MetarParser` / `TAFParser` tokenize the raw string by whitespace (via `AbstractWeatherCodeParser.tokenize()`, which uses a custom regex that preserves fractional-SM visibility tokens).
2. Each token is dispatched through a **`*CommandSupplier`** chain:
   - `CommonCommandSupplier` — wind, visibility, clouds, wind shear (shared by METAR and TAF)
   - `MetarCommandSupplier` — runway, temperature, altimeter (METAR-only)
   - `TAFCommandSupplier` — icing, turbulence (TAF-only)
   - `RemarkCommandSupplier` — all RMK section tokens
3. Each `Command` implements two methods:
   - `canParse(String input)` — determines if this command handles the token
   - `execute(AbstractWeatherContainer container, String part)` — mutates the container and returns `true`

**Parser class hierarchy:**

```
AbstractWeatherContainerParser<T, U>
  └── AbstractWeatherCodeParser<T>       (tokenization, airport lookup, flag parsing)
        ├── MetarParser                  (String → Metar)
        └── TAFParser                   (String[] lines → TAF)
              └── AbstractTAFTrendParser (shared trend parsing)
                    ├── FMTrendParser
                    └── ProbTrendParser
```

TAF trend parsers are instantiated by `TafTrendParserFactory` via `FactoryProvider`.

### 2.4 Airport Data — SPI Pattern

`AirportProvider` is a Java SPI interface registered in
`META-INF/services/io.github.mivek.provider.airport.AirportProvider`.

Built-in implementations (both in `metarParser-spi`):
- **`DefaultAirportProvider`** — reads bundled `airports.dat` / `countries.dat` CSV files; uses double-checked locking for lazy initialization.
- **`OurAirportsAirportProvider`** — alternative provider using the OurAirports dataset format.

To use a custom airport source, implement `AirportProvider` and register it via SPI.

### 2.5 Weather Retrieval — Strategy Pattern

`WeatherProvider` (interface in `io.github.mivek.service.provider`) decouples HTTP retrieval from
parsing. Built-in implementations:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mivek/MetarParser](https://github.com/mivek/MetarParser) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
