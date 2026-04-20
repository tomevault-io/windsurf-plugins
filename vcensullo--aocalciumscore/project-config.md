---
trigger: always_on
description: **Project Name:** AoCalciumScore (formerly SlicerCaScore)
---

# AoCalciumScore - Claude Development Documentation

## 📋 Project Overview

**Project Name:** AoCalciumScore (formerly SlicerCaScore)
**Version:** 3.0
**Author:** Vittorio Censullo
**Institution:** AITeRTC - Associazione Italiana Tecnici di Radiologia Esperti in TC
**Year:** 2025
**License:** MIT
**Repository:** https://github.com/vcensullo/AoCalciumScore

### Purpose
Professional 3D Slicer extension for automated quantification of aortic valve calcification using the Agatston scoring method. Provides guided workflow for calcium scoring with sex-specific severity classification and professional PDF reporting.

---

## 🏗️ Architecture Overview

### File Structure
```
AoCaScore/
├── AoCaScore/
│   ├── AoCaScore.py              # Main plugin file (~4200 lines)
│   ├── CMakeLists.txt             # Module configuration
│   ├── Resources/
│   │   ├── Icons/
│   │   │   ├── AoCaScore.png     # Main icon
│   │   │   ├── SlicerCaScore.png # Legacy icon
│   │   │   ├── click_grow.png    # Point & Click method icon
│   │   │   ├── roi_box.png       # ROI Box method icon
│   │   │   └── brush.png         # Paint mode icon
│   │   └── UI/
│   │       └── SlicerCaScore.ui  # Qt UI definition
│   └── Testing/
│       ├── CMakeLists.txt
│       └── Python/
│           └── CMakeLists.txt
├── CMakeLists.txt                 # Extension configuration
├── LICENSE                        # MIT License
├── README.md                      # User documentation
├── AoCaScore.png                  # Extension icon
└── .gitignore                     # Git ignore rules
```

---

## 🔬 Technical Implementation

### Core Algorithm: Agatston Scoring Method

#### 1. **Minimum Lesion Area Standard**
**Location:** `AoCaScore.py` lines ~2345-2370

**Implementation:**
```python
MIN_AREA_MM2_STANDARD = 1.0  # mm² - fixed clinical standard

# Calculate minimum pixels needed to reach 1 mm² threshold
minPixelsFor1mm2 = int(np.ceil(MIN_AREA_MM2_STANDARD / sliceArea))
MIN_PIXELS_PER_SLICE = max(2, minPixelsFor1mm2)  # At least 2 pixels, but ≥1 mm²
MIN_AREA_MM2 = MIN_PIXELS_PER_SLICE * sliceArea  # Actual threshold applied
```

**Clinical Standard:**
- Minimum lesion area: **≥1 mm² per slice** (Agatston et al. 1990)
- Traditional criterion: ≥3 contiguous pixels (for ~0.5mm pixel spacing)
- Modern high-resolution CT: Adaptive pixel threshold based on actual pixel size
- Example: 0.377mm spacing → 2 pixels × 0.142 mm² = 0.284 mm² (too small!)
- Correct: ceil(1.0 / 0.142) = 8 pixels needed for 1 mm²

**Reference:** "Only contiguous voxels totaling ≥1 mm² are counted as lesions"

#### 2. **Density-Based Weighting**
**Location:** `AoCaScore.py` lines ~2400-2450

**Density Factors:**
- Factor 1: 130-199 HU
- Factor 2: 200-299 HU
- Factor 3: 300-399 HU
- Factor 4: ≥400 HU

**Formula:**
```
Agatston Score = Σ (Lesion Area × Density Factor)
```

#### 3. **Slice-by-Slice Processing**
**Location:** `AoCaScore.py` lines ~2300-2600

**Method:**
- 2D labeling per slice (NOT 3D volumetric)
- Automatic overlap detection for thick slices (e.g., 3mm thickness, 1.5mm increment)
- Per-slice density factor calculation
- Lesion area calculation in mm² per slice

**Validation:**
- Matches commercial software (Syngo.via)
- ±1.3% accuracy in clinical validation

---

## 📊 Sex-Specific Severity Classification

### Current Thresholds (JAHA 2024)
**Location:** `AoCaScore.py` lines ~2503-2520

**Women:**
- **Severe AS:** ≥1300 AU
- **Moderate AS:** 400-1300 AU
- **Mild:** 100-400 AU
- **Normal/Minimal:** <100 AU

**Men:**
- **Severe AS:** ≥2000 AU
- **Moderate AS:** 1000-2000 AU
- **Mild:** 100-1000 AU
- **Normal/Minimal:** <100 AU

### Clinical Reference
**Primary Source:**
Tastet L, Ali M, Pibarot P, et al. "Grading of Aortic Valve Calcification Severity and Risk Stratification in Aortic Stenosis." *J Am Heart Assoc.* 2024;13(15):e035605.

**Key Findings:**
- First sex-specific thresholds to differentiate mild vs moderate AS
- Multicenter validation
- Women: 1300 AU (lower threshold reflects different disease progression)
- Men: 2000 AU (higher threshold due to larger valve area)

**Previous Standard (Deprecated):**
- Pawade et al. 2018: Women ≥1377 AU, Men ≥2062 AU
- Updated October 2024 to JAHA 2024 values

---

## 🎨 User Interface Architecture

### Tab-Based Workflow
**Implementation:** Qt TabWidget with sequential enablement

**Tab 0: Settings**
- Dependency installation (reportlab)
- Threshold configuration (default: 130 HU)
- Company logo and description for PDF reports
- Settings persistence using QSettings

**Tab 1: Setup View**
- CT volume selection
- Layout optimization
- Automatic tab 2 enablement after volume selection

**Tab 2: Calcium Segmentation**
Three segmentation methods:

1. **Point and Click** (Recommended)
   - One-click intelligent segmentation
   - Automatic region growing from seed point
   - Threshold-guided expansion

2. **ROI Box**
   - Manual box placement around aortic valve
   - Region-based threshold segmentation
   - Precise control for complex cases

3. **Paint Mode**
   - Manual painting with threshold-guided brush
   - Fine-grained control
   - Useful for difficult anatomy

**Automatic noise filtering:** Lesions < 1 mm² removed

**Tab 3: Results & Analysis**
- Agatston Score calculation
- Quantitative results display:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vcensullo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
