---
trigger: always_on
description: Comprehensive FHIR (Fast Healthcare Interoperability Resources) software development assistant. Use when working with FHIR APIs, implementations, or healthcare data exchange. Supports FHIR R4, R4B, R5, Implementation Guides (IGs), FHIR Shorthand (FSH) authoring, SUSHI, GoFSH, validation, terminology, and SMART on FHIR. Ideal for building FHIR servers, clients, validators, IG authors, or healthcare applications that need to process FHIR resources.
---


# FHIR Software Development Skill

Expert guidance for building robust FHIR (Fast Healthcare Interoperability Resources) software systems with comprehensive package management, spec knowledge, and development workflows.

## Core Architecture

### 1. Package/Specification Management

**Local FHIR Package Cache:**
- Use `@fhir/package-loader` or equivalent for TypeScript/Node.js environments
- For Python: `fhir-package-loader` or custom implementation using `requests` + `json`
- Cache strategy: `~/.fhir/packages/` with version-specific directories
- Support packages: `hl7.fhir.r4.core`, `hl7.fhir.r5.core`, Implementation Guides

**Package Resolution Pattern:**
```typescript
// Load and cache FHIR packages
async function loadFhirPackage(packageId: string, version?: string) {
  const cacheDir = path.join(os.homedir(), '.fhir', 'packages', packageId, version || 'current');
  if (await fs.pathExists(cacheDir)) return loadFromCache(cacheDir);
  
  const packageData = await downloadPackage(packageId, version);
  await cachePackage(cacheDir, packageData);
  return packageData;
}
```

**Document Index Structure:**
Build searchable index from package contents:
- StructureDefinitions (resources, profiles, extensions)
- SearchParameters (for API implementation)  
- ValueSets and CodeSystems (terminology)
- OperationDefinitions (custom operations)
- CapabilityStatements (server capabilities)
- Example instances

### 2. Development Workflows

#### FHIR Resource Modeling
```python
# Use Pydantic for FHIR resource modeling in Python
from pydantic import BaseModel, Field, field_validator
from typing import Optional, List, Literal
from enum import Enum
import re

class PatientGender(str, Enum):
    MALE = "male"
    FEMALE = "female"
    OTHER = "other"
    UNKNOWN = "unknown"

class Patient(BaseModel):
    resourceType: Literal["Patient"] = "Patient"
    id: Optional[str] = None
    active: Optional[bool] = None
    name: Optional[List[dict]] = None
    gender: Optional[PatientGender] = None
    birthDate: Optional[str] = None

    @field_validator('birthDate')
    @classmethod
    def validate_birthdate(cls, v):
        if v and not re.match(r'^\d{4}-\d{2}-\d{2}$', v):
            raise ValueError('Invalid date format, must be YYYY-MM-DD')
        return v

    class Config:
        extra = "allow"  # Allow additional FHIR elements
```

#### FHIR Server Implementation Patterns

**FastAPI + Pydantic (Python):**
```python
from fastapi import FastAPI, HTTPException
from fhir.resources.patient import Patient

app = FastAPI()

@app.post("/Patient", response_model=Patient)
async def create_patient(patient: Patient):
    # Validate against FHIR spec
    patient.validate()
    # Store in database
    saved_patient = await db.save_patient(patient)
    return saved_patient

@app.get("/Patient/{patient_id}")
async def get_patient(patient_id: str):
    patient = await db.get_patient(patient_id)
    if not patient:
        raise HTTPException(404, "Patient not found")
    return patient
```

**Express + FHIR TypeScript (Node.js):**
```typescript
import express from 'express';
import { Patient, Bundle } from 'fhir/r4';

const app = express();

app.post('/Patient', (req, res) => {
  const patient: Patient = req.body;
  
  // Validate resource type and required fields
  if (patient.resourceType !== 'Patient') {
    return res.status(400).json({
      resourceType: 'OperationOutcome',
      issue: [{
        severity: 'error',
        code: 'invalid',
        details: { text: 'Invalid resource type' }
      }]
    });
  }
  
  // Process and store
  const savedPatient = db.savePatient(patient);
  res.status(201).json(savedPatient);
});
```

### 3. FHIR Validation and Quality

#### Profile Validation
```python
from fhir.resources.core.fhirabstractmodel import FHIRAbstractModel
from fhir.resources import get_fhir_model_class
import json

def validate_against_profile(resource_data: dict, profile_url: str) -> bool:
    """Validate FHIR resource against specific profile"""
    try:
        # Load profile from package cache
        profile = load_structure_definition(profile_url)

        # Validate using fhir.resources - dynamically get the resource class
        resource_type = resource_data.get('resourceType')
        resource_class = get_fhir_model_class(resource_type)
        resource = resource_class(**resource_data)

        # Additional profile-specific validation
        return validate_profile_constraints(resource, profile)
    except Exception as e:
        print(f"Validation error: {e}")
        return False
```

#### Terminology Validation
```python
def validate_coding(coding: dict, value_set_url: str) -> bool:
    """Validate coding against ValueSet"""
    value_set = load_value_set(value_set_url)
    
    # Check if code exists in ValueSet expansion
    for concept in value_set.get('expansion', {}).get('contains', []):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PhenoML/ClaudeFHIRSkill](https://github.com/PhenoML/ClaudeFHIRSkill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
