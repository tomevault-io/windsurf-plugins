---
trigger: always_on
description: **Vote Match** is a Python CLI tool for processing voter registration records for GIS applications. It:
---

# Copilot Instructions for Vote Match

## Project Overview

**Vote Match** is a Python CLI tool for processing voter registration records for GIS applications. It:

1. **Loads** voter data from CSV files into PostGIS database
2. **Geocodes** addresses via multiple services (Census, Nominatim, Mapbox, Geocodio, Google Maps, Photon)
3. **Validates** addresses via USPS API
4. **Compares** voter district assignments against spatial boundaries (GeoJSON/Shapefile)
5. **Generates** interactive Leaflet maps with district mismatches
6. **Uploads** maps to Cloudflare R2 storage

### Key Use Case

Identify voters whose registered district (per Secretary of State) does not match their physical location (per GIS boundaries). Supports **all 15 district types** in Georgia voter data: congressional, state senate, state house, county commission, judicial, school board, city council, municipal school board, water board, super council, super commissioner, super school board, fire district, municipality, county precinct.

---

## Language & Runtime

- **Python 3.13+** (required)
- **Package manager**: `uv` ONLY (never `python -m pip`, `pip`, or `poetry`)
- **Build**: `pyproject.toml` with hatchling backend
- **CLI framework**: Typer with Rich for styled terminal output
- **Dependencies**: pandas, loguru, Jinja2, SQLAlchemy 2.0, GeoAlchemy2, Alembic, httpx

### Installation Commands

```bash
# Install dependencies
uv sync

# Run CLI
uv run vote-match --help

# Run tests
uv run pytest

# Lint/format
uv run ruff check
uv run ruff format
```

---

## Code Style & Linting

### Ruff Configuration

- **Line length**: 100 characters (hard limit)
- **Target**: Python 3.13
- **Config**: `[tool.ruff]` in `pyproject.toml`

### Style Guidelines

```python
# ✅ CORRECT: Python 3.13 union syntax
def get_voter(id: str | None) -> Voter | None:
    ...

# ❌ WRONG: Optional type (outdated)
from typing import Optional
def get_voter(id: Optional[str]) -> Optional[Voter]:
    ...

# ✅ CORRECT: String column for district IDs
district = Column(String, nullable=True)  # Preserves "026" vs "26"

# ❌ WRONG: Integer column (loses leading zeros)
district = Column(Integer, nullable=True)  # "026" becomes 26
```

### Import Order

1. Standard library (`os`, `pathlib`, `json`)
2. Third-party (`typer`, `loguru`, `sqlalchemy`)
3. Local modules (`vote_match.models`, `vote_match.processing`)

---

## Database & ORM

### Technologies

- **Database**: PostgreSQL with PostGIS extension
- **ORM**: SQLAlchemy 2.0+ (declarative style with `declarative_base()`)
- **Geometry**: GeoAlchemy2 for PostGIS columns
- **Migrations**: Alembic for schema versioning
- **CRS**: EPSG:4326 (WGS84) for all spatial data

### Key Models

```python
# src/vote_match/models.py

Base = declarative_base()

# District type key → Voter column name mapping
DISTRICT_TYPES: dict[str, str] = {
    "county": "county",
    "congressional": "congressional_district",
    "state_senate": "state_senate_district",
    "state_house": "state_house_district",
    "county_commission": "county_commission_district",
    "school_board": "school_board_district",
    # ... 15 types total
}

class Voter(Base):
    """Voter registration record with geocoding results."""
    __tablename__ = "voters"
    
    # All columns are String (not Integer) to preserve leading zeros
    voter_registration_number = Column(String, primary_key=True)
    congressional_district = Column(String, nullable=True)  # "026" not 26
    residence_zipcode = Column(String, nullable=True)  # "30901" not 30901
    
    geom = Column(Geometry("POINT", srid=4326), nullable=True)

class GeocodeResult(Base):
    """Multi-service geocoding results."""
    __tablename__ = "geocode_results"
    
    voter_id = Column(String, ForeignKey("voters.voter_registration_number"))
    service_name = Column(String(50))  # 'census', 'nominatim', etc.
    status = Column(String(20))  # 'exact', 'interpolated', 'approximate', 'no_match', 'failed'
    
class DistrictBoundary(Base):
    """Universal district boundary model (replaces legacy CountyCommissionDistrict)."""
    district_type = Column(String(50), nullable=False)  # Key from DISTRICT_TYPES
    geom = Column(Geometry("MULTIPOLYGON", srid=4326))

class VoterDistrictAssignment(Base):
    """Tracks spatial district assignments and mismatches."""
    voter_id = Column(String, ForeignKey("voters.voter_registration_number"))
    district_type = Column(String(50))
    spatially_assigned_district_id = Column(String(10))
    registered_district_id = Column(String(10))
    is_mismatch = Column(Boolean)
```

### Critical Rules

1. **ALL string columns must be `String`** (never `Integer`) to preserve leading zeros in:
   - Voter registration numbers
   - District IDs ("026", "07", "003")
   - ZIP codes ("30901", "01234")
   - FIPS codes ("13021", "01001")

2. **All geometry must be EPSG:4326** (WGS84 lat/lon)
   - Reproject shapefiles/GeoJSON before import
   - Use `geopandas.to_crs("EPSG:4326")` for conversions

3. **Batch operations commit in chunks** (default 1000 records)
   ```python
   for i, batch in enumerate(chunks(records, 1000)):
       session.bulk_save_objects(batch)
       session.commit()
   ```

4. **Use PostgreSQL upserts** for bulk loads:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kerryhatcher) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
