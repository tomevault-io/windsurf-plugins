---
trigger: always_on
description: ZUGFeRD / Factur-X / XRechnung on top of `@jasy/pdf`: one `Invoice` object in, a PDF/A-3 with the
---

# @jasy/e-invoice — CLAUDE.md

ZUGFeRD / Factur-X / XRechnung on top of `@jasy/pdf`: one `Invoice` object in, a PDF/A-3 with the
EN 16931 XML embedded out, in both permitted syntaxes (CII and UBL). This is the strategic prize of
the whole project and it is **legally critical** - a wrong invoice costs a customer money, not a
pixel. Read this before touching anything here.

## The shape

`src/invoice.ts` is the model and the public API; every field carries its `BT-` / `BG-` code in its
own doc comment. `compute.ts` derives every total and the VAT breakdown - the user never types a sum,
so the BR-CO total rules cannot fail. `cii.ts` and `ubl.ts` write the two syntaxes from the same
model and the same computed figures. `template.ts` draws the paper. `profile-check.ts` is the
pre-flight (`en16931Problems`, `xrechnungProblems`): plain sentences before the file exists. `render.ts`
assembles the PDF/A-3. `skonto.ts`, `allowance.ts`, `attachment.ts` each own ONE derived thing.

**Provide inputs, we compute the maths.** Anything derivable is derived, in exactly one place, and
every consumer (totals, CII, UBL, paper, the CLI reader) reads that place. `acAmount` resolves an
allowance whether it was stated as a sum or as `baseAmount` + `percent`; `resolveDiscounts` turns a
Skonto tier into its deadline and amounts. Four separate `base * percent / 100` expressions is how
the printed figure and the billed figure start disagreeing.

## The question "do we cover everything?" has exactly one answer

**`COVERAGE.md`** in this directory (gitignored - ours, not shipped). Every business term of the
standard, read out of the source, with the counts DERIVED from the rows. It exists because that
question was answered "yes" five times from memory while `invoice.ts`'s own header listed the
deferred groups. Never answer it from memory again; update the register and read the number.

The term LIST itself was compiled from knowledge and then cross-checked against the Peppol BIS 3.0
syntax binding (2026-09-11): two address-line-3 terms were missing and are in now; the register and
Peppol name the same business terms. The crawl is one command away if the list ever changes.

Deferred on purpose, documented in `invoice.ts`: tax representative (BG-11/12), payment card (BG-18),
item attributes (BG-32), item classification (BT-158). Left for 1.1 on the board: BT-147/148 (list
price with discount - a BG-27 line allowance already expresses the same money), BT-7/8 (VAT point
date), BT-15/16 (receiving/despatch advice).

## The schemas are the authority, not memory

`schema/cii` (Factur-X 1.08) and `schema/ubl` (UBL 2.1) are vendored. **CII is sequence-bound**: a
correct element in the wrong slot is an invalid file, and the business-rule validator does not check
sequence. `tests/cii-order.test.ts` and `tests/ubl-order.test.ts` DERIVE the expected order from the
XSDs, never from a hand-typed list; they caught `AccountingCost` before `BuyerReference` and
`OriginatorDocumentReference` before `ContractDocumentReference`, both of which every other check
waved through. Before placing a new element, read its parent's `complexType` in the XSD:

```bash
awk '/complexType name="HeaderTradeSettlementType"/,/<\/xs:complexType>/' schema/cii/*Reusable*.xsd | grep -o 'name="[A-Za-z]*"'
```

Things the XSD settled that would otherwise have been guesses: BT-26 is `qdt:` not `udt:` (the only
qualified type we emit); `TaxTotalAmount` has `maxOccurs="2"`, which IS how BT-110 and BT-111 are
told apart (same tag, different `currencyID`); `RoundingAmount` sits before `GrandTotalAmount` yet
moves the PAYABLE, not the total - follow the semantics, not the sequence.

**The two syntaxes disagree constantly.** BT-13/BT-14 are two CII elements but one UBL
`OrderReference`; BT-17/BT-18/BG-24 share one CII element told apart by `TypeCode` (50 / 130 / 916)
but are three different UBL elements; BT-21 has a CII element and NO UBL element (the binding
prefixes the note with `#CODE#`); BG-19 is scattered over three CII blocks and kept in two UBL ones;
BT-90 hangs on the UBL SELLER party with `schemeID="SEPA"`, in the same element as BT-29.

## Traps

**Skonto is not an allowance.** A discount conditional on early payment must not reduce the total,
because at issue nobody knows whether it will be taken. It lives in BT-20 as `#SKONTO#TAGE=n#PROZENT=n.nn#`
beneath the human terms. Entered as an allowance - the obvious workaround - it deducts immediately:
schema-valid, factually false, invisible to every validator. `profile-check` names that mistake by
matching the allowance's reason text. The rationale lives in `skonto.ts` ONCE; do not copy it.

**A non-EUR invoice used to go out with no Euro tax figure and no warning.** Same shape as Skonto:
input accepted, deficient document, silence. `xrechnungProblems` now demands BT-6/BT-111 when the
currency is not EUR. The amount is NEVER derived - which exchange rate applies is a tax question.

**Attachment names collide silently.** A supporting document named `factur-x.xml` lands in the same
name tree as the invoice XML; a reader picks whichever it finds first, and veraPDF calls the file
compliant (measured). Reserved names and duplicates are REFUSED, never renamed - the name is written

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jasy-pdf/jasy](https://github.com/jasy-pdf/jasy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
