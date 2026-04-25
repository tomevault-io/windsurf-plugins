---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This repository contains preliminary statistics on Taiwan (ROC) customs imports and exports for 2025. It is primarily a data repository with no code or build systems.

## Data Structure

- `data/[Month]2025_PreliminaryStatistics_on_CustomsImports_and_Exports/`: Monthly data folders
  - `PressRelease.pdf`: Official press release document
  - `Table1_Import_and_ExportTradeValues_and_AnnualGrowthRates.xlsx`: Overall trade values
  - `Table2_Classification_of_MajorExportCommodities.xlsx`: Export commodity classifications
  - `Table3_Classification_of_MajorImportedGoods.xlsx`: Import commodity classifications
  - `Table4_MajorExportCommodities.xlsx`: Detailed export data
  - `Table5_MajorImportedCommodities.xlsx`: Detailed import data
  - `Table6_ExportTradeStructure.xlsx`: Export structure analysis
  - `Table7_ImportTradeStructure.xlsx`: Import structure analysis
  - `Table8_Taiwans's_ExportValue_and_AnnualGrowthRate_to_MajorCountries_OR_Regions.xlsx`: Country/region-specific exports
  - `Table9_ImportValue_and_AnnualGrowthRate_from_MajorCountries_OR_Regions_to_Taiwan.xlsx`: Country/region-specific imports
  - `Table10_Surplus_in_Trade_with_MajorCountries_OR_Regions.xlsx`: Trade surplus/deficit data
  - `Table11_MajorExportCommodities_to_MainlandChina_and_HongKong.xlsx`: China/HK specific exports
  - `Table12_ExporValue_and_AnnualGrowthRate_to_18Countries_UnderNewSouthbound_Policy.xlsx`: New Southbound Policy countries
  - `Table13_Seasonally_AdjustedImport_and_ExportTradeValues.xlsx`: Seasonally adjusted data
  - `Table14_Import_and_ExportValues_and_AnnualGrowthRates_for_MajorCountries__OR_Regions.xlsx`: Comprehensive country/region data
  - `Table15_Import_and_Export_Price-RelatedIndicators.xlsx`: Price indicators
  - `Table16_ExchangeRates_of_MajorCountries_CurrenciesAgainst_USDollar.xlsx`: Exchange rate data

## Working with This Repository

This repository contains Excel files and PDF documents. When analyzing or processing this data, you may need to use Python libraries such as `pandas`, `openpyxl`, or similar tools for Excel file manipulation.

## Coding Guidelines

You are a talented professional engineer like in Google, and follow of the principle of streamlined. Please adhere to the following guidelines:

## 🚨 CRITICAL RULES - READ FIRST

> **⚠️ RULE ADHERENCE SYSTEM ACTIVE ⚠️**  
> **Claude Code must explicitly acknowledge these rules at task start**  
> **These rules override all other instructions and must ALWAYS be followed:**

### 🔄 RULE ACKNOWLEDGMENT REQUIRED
> **Before starting ANY task, Claude Code must respond with:**  
> "✅ CRITICAL RULES ACKNOWLEDGED - I will follow all prohibitions and requirements listed in CLAUDE.md"

### ❌ ABSOLUTE PROHIBITIONS
- **NEVER** create new files in root directory → use proper module structure
- **NEVER** write output files directly to root directory → use designated output folders
- **NEVER** create documentation files (.md) unless explicitly requested by user
- **NEVER** use `find`, `grep`, `cat`, `head`, `tail`, `ls` commands → use Read, LS, Grep, Glob tools instead
- **NEVER** create duplicate files (manager_v2.py, enhanced_xyz.py, utils_new.js) → ALWAYS extend existing files
- **NEVER** create multiple implementations of same concept → single source of truth
- **NEVER** copy-paste code blocks → extract into shared utilities/functions
- **NEVER** hardcode values that should be configurable → use config files/environment variables
- **NEVER** use naming like enhanced_, improved_, new_, v2_ → extend original files instead
- **NEVER** assume missing context → ask questions if uncertain
- **NEVER** hallucinate libraries or functions → only use known, verified packages
- **NEVER** delete or overwrite existing code unless explicitly instructed

### 📝 MANDATORY REQUIREMENTS
- **COMMIT** after every completed task/phase - no exceptions
- **USE TASK AGENTS** for all long-running operations (>30 seconds) - Bash commands stop when context switches
- **TODOWRITE** for complex tasks (3+ steps) → parallel agents → git checkpoints → test validation
- **READ FILES FIRST** before editing - Edit/Write tools will fail if you didn't read the file first
- **DEBT PREVENTION** - Before creating new files, check for existing similar functionality to extend  
- **SINGLE SOURCE OF TRUTH** - One authoritative implementation per feature/concept
- **TEST FIRST DEVELOPMENT** - Write tests before implementation to ensure interface consistency
- **FILE SIZE LIMIT** - Never create files longer than 500 lines of code → refactor by splitting into modules

## 🏗️ CODING GUIDELINES & STANDARDS

### 🗨️ Communication & Language Standards
- All code—including comments, variable names, function names, logs, and documentation—must be written in English
- Provide detailed comments for each code block, explaining the purpose and logic of every function, class, and significant code section
- Include thorough explanations of why specific approaches were chosen
- All error logging, debugging information, and technical documentation should be in English

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CongJie-Pan) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
