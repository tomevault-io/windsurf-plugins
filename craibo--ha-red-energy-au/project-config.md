---
trigger: always_on
description: > Created: 2025-10-06
---

# Red Energy Breakdown Sensors Implementation Plan

> Created: 2025-10-06
> Status: Approved - Ready for Implementation
> Branch: breakdown-usage-and-cost-sensors

## Overview

This plan details the implementation of comprehensive cost and usage breakdown sensors for the Red Energy Home Assistant integration. The breakdown leverages the rich interval data provided by the Red Energy API to give users detailed insights into their energy consumption, generation, costs, and environmental impact.

---

## Sensor Categories

### **Core Sensors** (Always Enabled)
These sensors provide fundamental breakdown information and are created for all users:

1. **Daily Import Usage** - Current daily grid import (consumption)
2. **Daily Export Usage** - Current daily solar export (generation)
3. **Total Import Usage** (30-day) - Total grid import over period
4. **Total Export Usage** (30-day) - Total solar export over period
5. **Total Import Cost** - Total cost of imported energy (excl GST)
6. **Total Export Credit** - Total credit from exported energy
7. **Net Cost** - Actual cost after solar credits (Import Cost - Export Credit)

### **Advanced Sensors** (Opt-in via Configuration)
These sensors provide detailed analytics for power users:

8. **Peak Import Usage** - Import during PEAK tariff periods
9. **Offpeak Import Usage** - Import during OFFPEAK tariff periods
10. **Shoulder Import Usage** - Import during SHOULDER tariff periods
11. **Peak Export Usage** - Export during PEAK tariff periods
12. **Offpeak Export Usage** - Export during OFFPEAK tariff periods
13. **Shoulder Export Usage** - Export during SHOULDER tariff periods
14. **Max Demand** - Maximum demand in kW over period
15. **Max Demand Time** - Timestamp when max demand occurred
16. **Carbon Emissions** - Total carbon emissions in tonnes CO₂

---

## API Data Structure

From `red-energy-api-structure.md`, the Red Energy API provides:

```json
{
  "usageDate": "2025-09-06",
  "halfHours": [
    {
      "intervalStart": "2025-09-06T00:00:00+10:00",
      "primaryConsumptionTariffComponent": "OFFPEAK",
      "consumptionKwh": 0.128,
      "consumptionDollar": 0.03,
      "generationKwh": 0.0,
      "generationDollar": 0.0,
      "demandDetail": { ... }
    }
  ],
  "maxDemandDetail": { ... },
  "carbonEmissionTonne": 0.0057,
  "consumptionDollar": 1.65,
  "generationDollar": -0.3279
}
```

**Key Fields Available**:
- ✅ `primaryConsumptionTariffComponent` - PEAK/OFFPEAK/SHOULDER classification
- ✅ `consumptionKwh` and `generationKwh` - Separate import/export per interval
- ✅ `consumptionDollar` and `generationDollar` - Separate costs
- ✅ `maxDemandDetail` - Demand charge information
- ✅ `carbonEmissionTonne` - Daily carbon emissions

---

## Detailed Implementation Steps

### **Step 1: Update Constants** (`const.py`)

Add new sensor type constants:

```python
# Daily sensor types (CORE SENSORS)
SENSOR_TYPE_DAILY_IMPORT_USAGE = "daily_import_usage"
SENSOR_TYPE_DAILY_EXPORT_USAGE = "daily_export_usage"

# Total usage breakdown (CORE SENSORS)
SENSOR_TYPE_TOTAL_IMPORT_USAGE = "total_import_usage"
SENSOR_TYPE_TOTAL_EXPORT_USAGE = "total_export_usage"

# Cost breakdown (CORE SENSORS)
SENSOR_TYPE_TOTAL_IMPORT_COST = "total_import_cost"
SENSOR_TYPE_TOTAL_EXPORT_CREDIT = "total_export_credit"
SENSOR_TYPE_NET_COST = "net_cost"

# Time period import breakdown (ADVANCED SENSORS)
SENSOR_TYPE_PEAK_IMPORT_USAGE = "peak_import_usage"
SENSOR_TYPE_OFFPEAK_IMPORT_USAGE = "offpeak_import_usage"
SENSOR_TYPE_SHOULDER_IMPORT_USAGE = "shoulder_import_usage"

# Time period export breakdown (ADVANCED SENSORS)
SENSOR_TYPE_PEAK_EXPORT_USAGE = "peak_export_usage"
SENSOR_TYPE_OFFPEAK_EXPORT_USAGE = "offpeak_export_usage"
SENSOR_TYPE_SHOULDER_EXPORT_USAGE = "shoulder_export_usage"

# Demand and environmental (ADVANCED SENSORS)
SENSOR_TYPE_MAX_DEMAND = "max_demand"
SENSOR_TYPE_MAX_DEMAND_TIME = "max_demand_interval_start"
SENSOR_TYPE_CARBON_EMISSION = "carbon_emission_tonne"

# Time period values (from API)
TIME_PERIOD_PEAK = "PEAK"
TIME_PERIOD_OFFPEAK = "OFFPEAK"
TIME_PERIOD_SHOULDER = "SHOULDER"
```

**File Modified**: `custom_components/red_energy/const.py`

---

### **Step 2: Enhance API Data Normalization** (`api.py`)

#### **Modify `_normalize_usage_entry()` Method**

Current implementation sums only `consumptionKwh`. Enhanced version extracts comprehensive breakdown:

**New Data Structure Returned**:
```python
{
    # Backward compatibility (keep existing)
    "date": "2025-09-06",
    "usage": 10.5,      # Net usage (import - export) for compatibility
    "cost": 1.32,       # Net cost for compatibility
    "unit": "kWh",
    
    # New breakdown fields
    "import_usage": 12.5,           # Sum of all consumptionKwh
    "export_usage": 2.0,            # Sum of all generationKwh
    "import_cost": 1.65,            # consumptionDollar (excl GST)
    "export_credit": 0.33,          # abs(generationDollar)
    "net_cost": 1.32,               # import_cost - export_credit
    
    # Time period import breakdowns
    "peak_import_usage": 5.0,
    "offpeak_import_usage": 6.0,
    "shoulder_import_usage": 1.5,
    
    # Time period export breakdowns
    "peak_export_usage": 0.5,
    "offpeak_export_usage": 1.0,
    "shoulder_export_usage": 0.5,
    
    # Demand and environmental
    "max_demand_kw": 4.5,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [craibo/ha-red-energy-au](https://github.com/craibo/ha-red-energy-au) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
