---
trigger: always_on
description: Specific to `darvis/ubl-peppol`. The shared conventions are in `~/Sites/Packages/CLAUDE.md`; only what differs or what this package adds is written here.
---

# CLAUDE.md

Specific to `darvis/ubl-peppol`. The shared conventions are in `~/Sites/Packages/CLAUDE.md`; only what differs or what this package adds is written here.

## Overview

Builds UBL 2.1 invoices and credit notes that pass PEPPOL BIS Billing 3.0 and EN 16931 validation, for the Netherlands and Belgium. It also validates European VAT numbers against VIES and company registration numbers such as the Dutch KvK number and the Belgian ondernemingsnummer, and can hand a finished document to a PEPPOL access point provider.

## Laravel is optional here

This is the one deliberate departure from the shared layout: `laravel/framework` is **not** a runtime requirement, only a dev dependency through Testbench. The package is a plain PHP library that happens to ship a Laravel layer.

Exactly four files may import `Illuminate\...`:

- `src/UblPeppolServiceProvider.php`
- `src/PeppolService.php`
- `src/Models/PeppolLog.php`
- `src/Console/CleanupPeppolLogsCommand.php`

`tests/Unit/StandaloneCoreTest.php` fails on an import anywhere else in `src/`, and it also checks that this list only names files that exist. If a file legitimately joins the Laravel layer, add it to the list in that test and say why in the pull request. Do not solve a failure by widening the rule.

`tests/Pest.php` therefore binds the Testbench `TestCase` only to `tests/Laravel`. Everything else runs without booting an application, which is both the point and the reason the suite is fast.

## Architecture

- `UblNlBis3Service` and `UblBeBis3Service` are separate classes on purpose: their checks and their method signatures differ: the Dutch `validate()` checks code formats and the NL-R rules, the Belgian one checks the totals. Never merge them behind a country flag. A fix in one is not automatically right in the other.
- Both builders build credit notes with the same four method names (`createCreditNoteDocument()`, `addCreditNoteHeader()`, `addBillingReference()`, `addCreditNoteLine()`), because a host app picks a builder per country and runs one code path over it: until 1.10.0 the Dutch builder lacked them and a Dutch credit note died with `Call to undefined method`. Never add a document level method to one builder under a name the other does not have. `<CreditNote>` has its own element order (`CREDIT_NOTE_ROOT_ELEMENT_ORDER`, copied from `UBL-CreditNote-2.1.xsd`): no `DueDate`, `AllowanceCharge` behind the exchange rates. Never sort a credit note with the invoice list.
- Elements are written in the order the UBL schema fixes. A document with correct values in the wrong order is rejected, and the receiver's error names the element, not the order, so this is the first thing to check when correct-looking XML fails. `UblNlBis3Service::arrangeInSchemaOrder()` sorts the children of `<Invoice>` when `generateXml()` runs, with a stable sort, so a document built in schema order comes out byte for byte as before; `tests/Unit/UblNlElementOrderTest.php` pins that. It matches on the node name, because `localName` is empty for an element made with `createElement()`. The Belgian builder only moves the totals in front of the lines.
- The builders write nothing the caller did not pass. Never hard code a value from the PEPPOL example files (`base-example.xml`) into a document: 1.9 and older wrote its `AccountingCost` and its supplier `RegistrationName` into every invoice, which a receiver books and stores as fact. An optional element gets its own `add...()` method (`addAccountingCost()`, BT-19).
- `PartyLegalEntity/CompanyID` (BT-30, BT-47) is the legal registration, not the VAT number. NL-R-003 and NL-R-005 test its `schemeID` for `0106` (KvK) or `0190` (OIN) on a Dutch party. The same goes for the customer's `PartyIdentification/cbc:ID` (BT-46): it is the seller's own name for the buyer, so it only carries the endpoint's scheme when `partyIdFitsScheme()` says so; PEPPOL-COMMON-R040 to R054 test the format of every identifier under a scheme, some of them fatally. Never write a value under scheme `0106` that is not a KvK number: the Schematron only checks the scheme, so nobody finds out until the receiver's bookkeeping does. The Dutch builder takes it through `addSupplierLegalRegistration()` and `addCustomerLegalRegistration()`; the party signatures could not change within 1.x. `tests/Unit/GeneratedValuesNlTest.php` and `GeneratedValuesBeTest.php` pin these values, each test named after its business term and rule.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ArvidDeJong/ubl-peppol](https://github.com/ArvidDeJong/ubl-peppol) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
