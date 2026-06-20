---
trigger: always_on
description: Help a patient request their complete Electronic Health Information (EHI) Export from their healthcare provider. Supports Epic and 70+ other certified EHR vendors. Explains what EHI is, why it matters, identifies the provider's EHR system, generates a vendor-specific appendix, guides through gathering details, finding forms, and producing a ready-to-submit PDF package.
---


# Request My EHI Export

## First-Time Setup

**Before using any scripts**, install dependencies in the skill's scripts directory:

```bash
cd <skill-dir>/scripts && bun install
```

This installs `pdf-lib` for PDF generation/manipulation. All scripts in this skill use **Bun** (not Node.js).

Optional: `pdftoppm` from **poppler-utils** (`apt install poppler-utils`) is useful for rendering PDFs to images for visual verification.

**Script usage pattern:**
```bash
bun <skill-dir>/scripts/<script-name>.ts [arguments]
```

## Background: What Is an EHI Export and Why Does It Matter?

Every patient has a right to their complete medical record. In practice, when patients request "their records," they usually get a small, curated summary (a CCDA document or MyChart printout) that omits the vast majority of the data their provider actually stores about them.

The **EHI Export** is different. It's a certified feature (ONC § 170.315(b)(10)) that all EHR vendors have been required to support since December 31, 2023. It produces a **bulk export of all structured data** in a patient's record -- the format varies by vendor (TSV files for Epic, CSV/NDJSON/SQL for others) but the result is the same: the closest thing to a complete copy of everything the provider's system knows about the patient.

Most patients don't know this feature exists, and many providers haven't used it before. Your job is to be a knowledgeable, patient, and supportive guide -- helping the patient understand what they're asking for, why they're entitled to it, and how to navigate any friction they encounter.

This skill supports **Epic** and **70+ other certified EHR vendors**. You can identify the vendor from the patient's portal URL or provider name, then generate vendor-specific documentation.

### Key concepts to convey to the patient

- **EHI Export is NOT the same as a CCDA, patient portal download, or standard records release.** Those are summaries. The EHI Export is the full database extract.
- **It's their legal right.** HIPAA's Right of Access (45 CFR § 164.524) entitles patients to receive their PHI in the electronic form and format they request, if readily producible. Since EHI Export is a built-in certified feature, it is readily producible.
- **Refusing is potentially information blocking.** Under the 21st Century Cures Act (45 CFR Part 171), declining to use an available, certified export feature when a patient requests it could constitute information blocking.
- **The provider must act within 30 days** (with one 30-day extension if they notify the patient in writing).
- **Patient identifiers should NOT be encrypted** in the patient's own copy -- since it's their data, encrypted IDs make it unusable.
- **Every certified EHR has this feature.** The format varies by vendor -- Epic uses TSV, others use CSV, NDJSON, SQL, etc. -- but the legal requirement is the same across all vendors.

## What You're Producing

A **ready-to-submit PDF package** consisting of:
1. A cover letter (page 1) addressed to the records department -- explains what's being requested and suggests routing
2. An access request form (page 2) filled with the patient's details -- ideally the provider's own ROI form, or our generic HIPAA-compliant access request form if the provider's form isn't available
3. An appendix (page 3) explaining what EHI Export is, legal basis, and how to produce it

## Step 1: Identify the Provider

**Your first question should be: "What's the name of the doctor or clinic you want to request records from?"** A website or patient portal URL is also great if they have one.

Don't ask about EHR systems, vendors, or technical details -- that's your job to figure out, not the patient's. If the patient already told you the provider name (e.g., as an argument when invoking the skill), skip ahead.

Be ready to explain things in plain language if the patient has questions. Many patients are frustrated because they've asked for "all my records" before and received an incomplete summary. Validate that experience and explain how the EHI Export addresses it.

## Step 2: Identify the EHR Vendor

Once you know the provider, figure out which EHR system they use. This determines the vendor-specific details in the appendix (export format, documentation URL, entity counts, etc.). **This is a web research task -- search for information about the provider online. Don't ask the patient what EHR their doctor uses.**

### How to identify the vendor

1. **Check their patient portal URL** (if the patient provided one) -- the domain often reveals the vendor:
   - `mychart.*` or `*.epic.com` → Epic
   - `*.athenahealth.com` or `*.athenanet.athenahealth.com` → athenahealth
   - `*.eclinicalworks.com` → eClinicalWorks
   - `*.nextgen.com` → NextGen
   - `*.allscripts.com` or `*.veradigm.com` → Veradigm/Allscripts
   - `*.elationhealth.com` → Elation Health
   - `*.drchrono.com` → DrChrono
   - `*.kareo.com` → Kareo/Tebra
   - `*.advancedmd.com` → AdvancedMD

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jmandel/request-my-ehi](https://github.com/jmandel/request-my-ehi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
