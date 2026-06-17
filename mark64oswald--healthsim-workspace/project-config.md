---
trigger: always_on
description: HealthSim generates realistic synthetic healthcare data for testing EMR systems, claims processing, pharmacy benefits, and analytics. Use for ANY request involving: (1) synthetic patients, clinical data, or medical records, (2) healthcare claims, billing, or adjudication, (3) pharmacy prescriptions, formularies, or drug utilization, (4) HL7v2, FHIR, X12, or NCPDP formatted output, (5) healthcare testing scenarios or sample data generation.
---


# HealthSim - Synthetic Healthcare Data Generation

## Overview

HealthSim generates realistic synthetic healthcare data through natural conversation. Rather than writing code or configuration files, describe what you need and Claude generates appropriate data.

**Products:**

| Product | Domain | What It Generates | Status |
|---------|--------|-------------------|--------|
| **PatientSim** | Clinical/EMR | Patients, encounters, diagnoses, procedures, labs, vitals, medications | Active |
| **MemberSim** | Payer/Claims | Members, professional claims, facility claims, payments, accumulators | Active |
| **RxMemberSim** | Pharmacy/PBM | Prescriptions, pharmacy claims, formularies, DUR alerts, prior auths | Active |
| **TrialSim** | Clinical Trials | Studies, sites, subjects, visits, adverse events, efficacy, CDISC output | Active |
| **PopulationSim** | Demographics/SDOH | Population profiles, cohort specifications, health disparities, SVI/ADI analysis | Active |
| **NetworkSim** | Provider Networks | Providers, facilities, pharmacies, networks, benefit structures | Active |

## Quick Start

### Generate Clinical Data

**Request:** "Generate a 65-year-old diabetic patient with hypertension"

Claude will produce a patient with:
- Demographics (age 65, realistic name/address)
- Diagnoses (E11.9 Type 2 diabetes, I10 hypertension)
- Medications (metformin, lisinopril)
- Labs (A1C, BMP with values in expected ranges)
- Comorbidities (likely hyperlipidemia, possible obesity)

### Generate Claims

**Request:** "Create a professional claim for an office visit"

Claude will produce:
- Claim header (provider NPI, member ID, service date)
- Service lines (CPT 99213/99214, charges)
- Diagnoses (ICD-10 codes)
- Adjudication (allowed, paid, patient responsibility)

### Generate Pharmacy Data

**Request:** "Generate a pharmacy claim that triggers a drug interaction alert"

Claude will produce:
- Prescription details (NDC, quantity, days supply)
- Pharmacy claim (BIN, PCN, cardholder ID)
- DUR alert (DD code, clinical significance, recommendation)
- Claim response (approved with warning or rejected)

## Scenario Skills

### PatientSim Scenarios

Load these for clinical data generation:

| Scenario | Use When | Key Elements |
|----------|----------|--------------|
| **ADT Workflow** | admission, discharge, transfer, patient movement | A01/A02/A03 events, bed management, census |
| **Diabetes Management** | diabetic, A1C, glucose, metformin, insulin | Disease progression, medication escalation, complications |
| **Heart Failure** | CHF, HFrEF, BNP, ejection fraction | NYHA classification, GDMT therapy, decompensation |
| **Chronic Kidney Disease** | CKD, eGFR, dialysis, nephrology | CKD staging, progression, comorbidities |
| **Sepsis/Acute Care** | sepsis, infection, ICU, critical | Sepsis criteria, antibiotic protocols, ICU stay |
| **Orders & Results** | lab order, radiology, ORM, ORU, results | Orders, specimens, lab panels, radiology reports |
| **ED Chest Pain** | chest pain, emergency, ACS, troponin | Risk stratification, HEART score, workup |
| **Elective Joint** | hip replacement, knee replacement, arthroplasty | Pre-op, surgery, recovery, PT |
| **Maternal Health** | pregnancy, prenatal, L&D, postpartum | Prenatal visits, GDM, preeclampsia, delivery |
| **Oncology** | cancer, tumor, chemotherapy, breast/lung/colorectal | Staging, treatment protocols, tumor markers |

See: [skills/patientsim/](skills/patientsim/) for detailed skills

### MemberSim Scenarios

Load these for claims and payer data:

| Scenario | Use When | Key Elements |
|----------|----------|--------------|
| **Plan & Benefits** | plan, benefit plan, HMO, PPO, HDHP | Plan types, cost sharing, pharmacy tiers |
| **Enrollment & Eligibility** | enrollment, eligibility, 834, 270/271 | Member add/term, coverage verification, QLE |
| **Professional Claims** | office visit, 837P, physician claim | E&M coding, place of service, adjudication |
| **Facility Claims** | hospital, inpatient, 837I, DRG | Revenue codes, DRG assignment, LOS |
| **Prior Authorization** | prior auth, pre-cert, authorization | Request/response workflow, approval criteria |
| **Accumulator Tracking** | deductible, OOP, accumulator | Year-to-date tracking, family vs individual |
| **Value-Based Care** | quality measures, VBC, HEDIS | Attribution, measure compliance, incentives |
| **Behavioral Health** | mental health, psychiatry, SUD, therapy | Psychotherapy, medication management, PHP/IOP |

See: [skills/membersim/](skills/membersim/) for detailed skills

### RxMemberSim Scenarios

Load these for pharmacy and PBM data:

| Scenario | Use When | Key Elements |
|----------|----------|--------------|
| **Retail Pharmacy** | prescription fill, retail, copay | New/refill, pricing, patient pay |
| **Specialty Pharmacy** | specialty drug, biologics, hub | Limited distribution, PA, patient support |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mark64oswald/healthsim-workspace](https://github.com/mark64oswald/healthsim-workspace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
