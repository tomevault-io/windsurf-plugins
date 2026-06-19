---
trigger: always_on
description: Guidance for Claude Code working with this repository.
---

# CLAUDE.md

Guidance for Claude Code working with this repository.

## Build Commands

```bash
cargo build --release          # Build binary
cargo test                     # All tests
cargo test -p datasynth-core   # Specific crate
cargo test test_name           # Single test
cargo check                    # Check only
cargo fmt && cargo clippy      # Format + lint
cargo bench                    # Benchmarks
```

## CLI Usage

Binary: `datasynth-data` (at `target/release/datasynth-data`)

```bash
datasynth-data generate --demo --output ./output
datasynth-data init --industry manufacturing --complexity medium -o config.yaml
datasynth-data validate --config config.yaml
datasynth-data generate --config config.yaml --output ./output
kill -USR1 $(pgrep datasynth-data)  # Pause/resume (Unix)
```

### Group audit (v5.0+)

```bash
datasynth-data group manifest --config group.yaml --out manifest.json
datasynth-data group shard --manifest manifest.json --shard-id S_SIG_0001 --out ./shards/S_SIG_0001/
datasynth-data group aggregate --manifest manifest.json --shards-dir ./shards --out ./group_archive
datasynth-data group generate --config group.yaml --out ./group_archive   # in-process pipeline
```

## Server

```bash
cargo run -p datasynth-server -- --port 3000 --worker-threads 4
```

## Architecture

Rust workspace with 19 active crates:

```
datasynth-cli             → Binary (generate, validate, init, info, fingerprint, templates, data)
datasynth-server          → REST/gRPC/WebSocket server
datasynth-runtime         → EnhancedOrchestrator coordinates workflow (~30 phases)
datasynth-generators      → Data generators (JE, Document Flows, Subledgers, Anomalies, Audit)
datasynth-banking         → KYC/AML banking with fraud typologies
datasynth-ocpm            → OCEL 2.0 process mining
datasynth-fingerprint     → Privacy-preserving fingerprint extraction/synthesis
datasynth-standards       → Accounting/audit standards (IFRS, US GAAP, French GAAP, German GAAP, ISA, SOX, PCAOB)
datasynth-graph           → Graph export (PyTorch Geometric, Neo4j, DGL)
datasynth-graph-export    → Hypergraph → bulk node/edge export pipeline (RustGraph wire format; GH #218)
datasynth-eval            → Evaluation framework with auto-tuning
datasynth-config          → Configuration schema, validation, presets
datasynth-core            → Domain models, traits, distributions, resource guards, templates
datasynth-output          → Output sinks (CSV, JSON, Parquet)
datasynth-test-utils      → Test utilities
datasynth-audit-fsm       → YAML-driven audit state machines (engagements, blueprints)
datasynth-audit-optimizer → Risk-scoping / portfolio / Monte-Carlo / conformance analytics
datasynth-audit-triage    → Client-GL onboarding + fit-on-self triage (profile, calibrate, materiality, stress-MC, structural-change, relational fidelity)
datasynth-group           → Group audit simulation engine (manifest / shard / aggregate phases)
```

### Key Models (datasynth-core/src/models/)

| Category | Models |
|----------|--------|
| Accounting | JournalEntry, ChartOfAccounts, ACDOCA |
| Master Data | Vendor, Customer, Material, FixedAsset, Employee, EntityRegistry |
| Document Flow | PurchaseOrder, GoodsReceipt, VendorInvoice, Payment, SalesOrder, Delivery, CustomerInvoice, CustomerReceipt, DocumentReference |
| Sourcing (S2C) | SourcingProject, SupplierQualification, RfxEvent, SupplierBid, BidEvaluation, ProcurementContract, CatalogItem, SupplierScorecard, SpendAnalysis |
| Financial Reporting | FinancialStatement, FinancialStatementLineItem, CashFlowItem, ManagementKpi, Budget, BudgetLineItem |
| HR/Payroll | PayrollRun, PayrollLineItem, TimeEntry, ExpenseReport, ExpenseLineItem, BenefitEnrollment |
| Manufacturing | ProductionOrder, RoutingOperation, QualityInspection, InspectionCharacteristic, CycleCount, CycleCountItem, BomComponent, InventoryMovement |
| Sales | SalesQuote, QuoteLineItem |
| Bank Reconciliation | BankReconciliation, BankStatementLine, ReconcilingItem |
| Intercompany | IntercompanyRelationship, ICTransactionType, ICMatchedPair, TransferPricingMethod, GroupStructure, SubsidiaryRelationship, NciMeasurement |
| Subledger | AccountBalance, TrialBalance, AR*/AP*/FA*/Inventory* records, ARAgingReport, APAgingReport, DepreciationRun, InventoryValuation |
| FX/Close | FxRate, CurrencyTranslation, CurrencyTranslationResult, FiscalPeriod, AccrualEntry |
| Anomalies | AnomalyType, LabeledAnomaly, QualityIssue |
| Controls | InternalControl, ControlMapping, SoD |
| COSO Framework | CosoComponent, CosoPrinciple, ControlScope, CosoMaturityLevel |
| Vendor Network | VendorNetwork, VendorRelationship, VendorCluster, VendorLifecycleStage, VendorQualityScore, VendorDependency, SupplyChainTier |
| Customer Segment | SegmentedCustomer, CustomerValueSegment, CustomerLifecycleStage, CustomerNetworkPosition, CustomerEngagement, SegmentedCustomerPool |
| Tax | TaxJurisdiction, TaxCode, TaxLine, TaxReturn, TaxProvision, WithholdingTaxRecord, UncertainTaxPosition, TemporaryDifference, DeferredTaxRollforward, TaxRateReconciliation |
| Treasury | CashPosition, CashForecast, CashPool, CashPoolSweep, HedgingInstrument, HedgeRelationship, DebtInstrument, DebtCovenant |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mivertowski/SyntheticData](https://github.com/mivertowski/SyntheticData) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
