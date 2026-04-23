---
trigger: always_on
description: This repository manages the flock records, breeding program, and health tracking for a sheep operation at Manatee Creek in Florida. It serves three purposes:
---

# Manatee Creek Sheep — Project Guidelines

## Purpose

This repository manages the flock records, breeding program, and health tracking for a sheep operation at Manatee Creek in Florida. It serves three purposes:

1. **Flock management** — maintaining accurate records of every animal: identity, pedigree, breed composition, health, pen assignments, and lambing
2. **Breeding program** — tracking genetics across generations to make informed breeding decisions, maximize hybrid vigor, and manage inbreeding avoidance
3. **Image archive** — preserving photographs (spiral notebook notes, handwritten records, sheep photos) as primary source material

This is a family operation. Accuracy matters because real animals depend on correct records. Every data point affects breeding decisions, health interventions, and flock management.

---

## Theological Foundation

> *"The LORD is my shepherd; I shall not want."* — Psalm 23:1 (ESV)

**Soli Deo Gloria** — All work on this project is offered as a gift to God. We tend these sheep as stewards, not owners.

---

## Data Sources — Priority Order

The flock data comes from multiple sources. When sources conflict, use this priority:

| Priority | Source | Description |
|----------|--------|-------------|
| **1 (HIGHEST)** | Spiral notebook images (PNG files) | Mom's phone app notes — most current, most accurate reflection of what's on the ground |
| **2** | `flock_record_v2.xlsx` | Structured spreadsheet with pen rosters, lambing records, cross-references |
| **3** | `data.csv` | Historical records with breed percentages, weights, DOBs |
| **4** | Google Sheet | Breed composition calculations for potential offspring by pen |
| **5** | `Sheep_Breeding_DB_CURRENT_COPY.xlsx` | Mating plans, ram eligibility, breeding rules |

**Important aliases:**
- Mom writes "Amure" — this is **Azure**
- "Rock" = "Jerkface" = Awassi ram
- "Bt" = Broken Tail
- "Bsoe" / "Bsoed" are separate sheep (tags 31/32, switched)
- "GG" = Gigi
- Mc11 = Charlie's ram = tag 12
- Mc12 = 036 = Serendipity's baby ewe
- Mc01 = Little Daisy's baby (baby's baby) = tag 35
- NoriSon = ram in pen 5, tag 54

---

## Image Processing

### Critical Rule: Images Must Be Processed Before Reading

All original images (JPG and PNG) are 4032x3024px or 1320x2868px — both exceed Claude's 2000px API limit.

**Always use processed versions from `data/processed/`**

```bash
# Check image status
python3 scripts/process_images.py --status

# Process all oversized images
python3 scripts/process_images.py

# Process a single image
python3 scripts/process_images.py --file IMG_8560.JPG
```

### Image Categories

| Range | Type | Content |
|-------|------|---------|
| IMG_8560–8615 | JPG | Handwritten notebook pages (sheep check records with photos) |
| IMG_8616–8643 | PNG | Phone app notes (treatments, measurements, pen assignments, health notes) |

### Image Retention Policy

**NEVER delete any image.** Every image is a primary source document. Original handwritten records and photos of the sheep are irreplaceable.

---

## Database Schema

The flock database lives at `data/flock_database.json`. It contains:

- **sheep[]** — Every animal past and present with pedigree, breed composition, health, pen, status
- **pens{}** — Current pen assignments with breeding groups
- **lambing_records_2026[]** — 2026 lambing season records
- **breed_reference{}** — Classification of breeds as hair, wool, or dual-purpose

### Key Fields Per Sheep

| Field | Description | Required |
|-------|-------------|----------|
| `id` | Unique kebab-case identifier | Yes |
| `name` | Display name | Yes |
| `tag` | Ear tag number (if any) | No |
| `sex` | ram, ewe, ram_lamb, ewe_lamb, wether, unknown | Yes |
| `breed_composition` | Percentages of each breed | When known |
| `sire_id` / `dam_id` | References to parent sheep IDs | When known |
| `status` | alive, deceased, sold, unknown | Yes |
| `pen` | Current pen assignment | When known |
| `health` | FAMACHA scores, treatments, weak resistance flag | Ongoing |
| `confidence` | high, medium, low — how certain we are of the data | Yes |

### Breed Composition

Track percentages for all breeds present. Common breeds in this flock:

**Hair breeds:** Katahdin, Dorper, White Dorper, St Croix, Barbados Blackbelly (BBB), American Blackbelly (ABB), Wiltshire Horn
**Wool breeds:** Suffolk, Hampshire, Cotswold, Tunis, Gulf Coast Native (GCN)
**Dual-purpose:** St Augustine, Cracker, Awassi, East Friesian
**Other:** Jacob, Babydoll, Karakul

---

## Breeding Program Principles

### Long-Term Vision

The long-term goal is to produce sheep that are **MORE parasite resistant** while remaining **meaty and milky**. The flock draws from **22 breeds** — sheep whose former owners claimed no parasite issues. This has proved more or less true in most cases.

### Key Genetics Observations
- **Most parasite resistant:** Kelsier (Katahdin) — the gold standard in this flock
- **Most milky:** Awassi and Awassi crosses
- **Meatiest:** Dorper-Awassi cross
- **Best all-around:** Katahdin foundation with strategic crosses

### Selection Priorities

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jsschrstrcks1) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
