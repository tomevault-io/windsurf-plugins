---
trigger: always_on
description: > Ten plik zawiera kluczowe wytyczne architektoniczne, konwencje i zasady inżynieryjne obowiązujące w projekcie **Taran** — wysokowydajnym narzędziu do testów obciążeniowych napisanym w Rust.
---

# CLAUDE.md — Instrukcje dla Claude Code

> Ten plik zawiera kluczowe wytyczne architektoniczne, konwencje i zasady inżynieryjne obowiązujące w projekcie **Taran** — wysokowydajnym narzędziu do testów obciążeniowych napisanym w Rust.

---

## Przegląd projektu

**Taran** to CLI-first narzędzie do generowania obciążenia HTTP/gRPC/WebSocket, alternatywa dla JMeter/K6.
Kluczowe cechy: zero GC, async (Tokio), skryptowanie Rhai, HDR Histogram, real-time TUI dashboard.

Szczegółowy plan realizacji: `docs/PLAN.md`

---

## Struktura workspace (Cargo workspace)

```
taran/
├── Cargo.toml            # workspace root
├── taran-cli/            # binarka — punkt wejścia, parsowanie CLI (clap)
├── taran-core/           # silnik wykonawczy: VU, scheduler, load profiles
├── taran-config/         # parsowanie scenariuszy TOML/YAML (serde)
├── taran-metrics/        # zbieranie metryk: HDR Histogram, lock-free counters
├── taran-report/         # generowanie raportów: JSON, CSV, HTML, TUI
├── taran-script/         # silnik skryptowy Rhai — API dla scenariuszy
├── taran-protocols/      # klienty protokołów: HTTP, gRPC, WebSocket, TCP
└── docs/                 # dokumentacja projektu
```

### Zależności między crate'ami (kierunek →  = "zależy od")

```
taran-cli → taran-core → taran-config
                       → taran-metrics
                       → taran-protocols
                       → taran-script
taran-report → taran-metrics
taran-script → taran-protocols
```

**Reguła:** Crate'y niższego poziomu (config, metrics, protocols) NIE mogą zależeć od crate'ów wyższego poziomu (core, cli). Zależności płyną od góry do dołu.

---

## Clean Architecture

Projekt stosuje zasady Clean Architecture zaadaptowane do Rust:

### Warstwy (od wewnętrznej do zewnętrznej)

1. **Domain (taran-core)** — czysta logika biznesowa
   - Definicje trait'ów (`Protocol`, `LoadProfile`, `MetricsCollector`, `ScriptEngine`)
   - Struktury domenowe (`VirtualUser`, `Scenario`, `Step`, `Assertion`, `TestResult`)
   - Brak zależności od frameworków I/O — operuje na trait'ach
   - Nie importuje `tokio`, `reqwest`, `tonic` bezpośrednio

2. **Application (taran-core::engine)** — orkiestracja
   - `TestRunner` — uruchamia scenariusze, zarządza lifecycle VU
   - `Scheduler` — implementuje load profiles (constant, ramp, spike)
   - Operuje na abstrakcjach (trait objects / generics), nie na konkretnych typach

3. **Infrastructure (taran-protocols, taran-metrics, taran-config, taran-script)** — implementacje
   - Konkretne implementacje trait'ów z warstwy Domain
   - `HttpClient` impl `Protocol`, `GrpcClient` impl `Protocol`
   - `HdrMetricsCollector` impl `MetricsCollector`
   - `RhaiScriptEngine` impl `ScriptEngine`
   - `TomlConfigLoader` impl `ConfigLoader`

4. **Presentation (taran-cli, taran-report)** — interfejs użytkownika
   - CLI parsing (clap), TUI dashboard (ratatui)
   - Raporty HTML/JSON/CSV
   - Dependency injection — łączy wszystkie warstwy

### Reguła zależności

```
Presentation → Application → Domain ← Infrastructure
```

Infrastructure zależy od Domain (implementuje trait'y), NIE odwrotnie.
Presentation łączy wszystko przez dependency injection (konstruktory).

---

## Zasady SOLID w Rust

### Single Responsibility Principle (SRP)

- Każdy crate ma jedną odpowiedzialność (patrz struktura workspace)
- Każdy moduł (`mod`) ma wyraźnie zdefiniowany zakres
- Struktury mają jedną odpowiedzialność:
  - `Scheduler` — tylko harmonogram uruchamiania VU
  - `HttpClient` — tylko komunikacja HTTP
  - `HdrCollector` — tylko zbieranie metryk
- **NIE** twórz "god structs" łączących wiele odpowiedzialności

### Open/Closed Principle (OCP)

- Nowe protokoły dodawane przez implementację trait `Protocol`, bez modyfikacji `taran-core`
- Nowe load profiles przez implementację trait `LoadProfile`
- Nowe formaty raportów przez implementację trait `Reporter`
- Używaj enum dispatch (`enum_dispatch` crate) lub trait objects (`Box<dyn Protocol>`) do polimorfizmu

```rust
// ✅ Otwarte na rozszerzenie
pub trait Protocol: Send + Sync {
    async fn execute(&self, request: &Request) -> Result<Response>;
    fn protocol_name(&self) -> &str;
}

// Nowy protokół = nowy struct + impl, zero zmian w istniejącym kodzie
pub struct GraphqlClient { /* ... */ }
impl Protocol for GraphqlClient { /* ... */ }
```

### Liskov Substitution Principle (LSP)

- Wszystkie implementacje trait'ów muszą spełniać kontrakt trait'a bez niespodzianek
- Jeśli trait definiuje `async fn execute() -> Result<Response>`, każda implementacja musi zwracać sensowny `Response` lub `Error` — nigdy `panic!()`
- Testy integracyjne powinny być parametryzowane po trait'ach, nie po konkretnych typach

### Interface Segregation Principle (ISP)

- Trait'y powinny być małe i fokusowane
- **NIE** twórz "fat traits" z 20 metodami
- Preferuj kompozycję trait'ów:

```rust
// ✅ Segregowane traity
pub trait Connectable {
    async fn connect(&mut self) -> Result<()>;
    async fn disconnect(&mut self) -> Result<()>;
}

pub trait RequestExecutor {
    async fn execute(&self, req: &Request) -> Result<Response>;
}

pub trait Protocol: Connectable + RequestExecutor + Send + Sync {}


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Shaqal7) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
