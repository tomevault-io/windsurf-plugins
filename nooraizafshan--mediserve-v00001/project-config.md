---
trigger: always_on
description: AI coding agents should understand these architecture patterns before implementation.
---

# MediServe Copilot Instructions

AI coding agents should understand these architecture patterns before implementation.

## Project Overview

**MediServe** is a medical report management system for hospitals/diagnostic centers. It automates workflow from lab uploads → consultant verification → departmental approvals → QR-validated certificate generation.

Tech Stack: Next.js 14+ App Router, TypeScript, MongoDB/Mongoose, Cloudinary, Tailwind CSS

## Critical Business Rule: Department-Based Lab Records

**NEVER use generic test arrays.** Lab tests MUST be stored as department-nested attributes:

```typescript
// ✓ CORRECT
departments.medicine.HCV = { value: "Positive", fileUrl: "..." }
departments.ophthalmology.LeftEyeVision = "6/6"

// ✗ WRONG - DO NOT USE
tests: [{ name: "HCV", value: "Positive" }]
```

This rule applies to:
- `src/models/LabRecord.ts` (schema design)
- `src/app/api/lab-record/**` (all API routes)
- `src/components/LabUploadForm.tsx` (form submission)
- `src/lib/services/LabRecordService.ts` (business logic)

## Architecture Layers

### 1. Models (`src/models/`)
- `LabRecord.ts`: Department-nested schema with 4 departments (medicine, ophthalmology, surgery, psychiatry)
- `caseModel.ts`: Medical case workflow (pending → payment-pending → lab-complete → approved)
- `userModel.ts`: Role-based access (admin, deo, consultant, dms, ms, it-support)

**Pattern**: Mongoose schemas with strict mode enabled, timestamps, and indexing.

### 2. Database Connection (`src/lib/mongooseConnect.ts`)
- Global cached connection for Next.js serverless functions
- Connection pooling (5-10 connections)
- Graceful shutdown on SIGINT

**Pattern**: Always use `await connectDB()` before model operations.

### 3. Services (`src/lib/services/LabRecordService.ts`)
- Abstraction layer between API routes and models
- Static methods for all CRUD operations
- ObjectId validation on all inputs
- Error handling with descriptive messages

**Pattern**: Use DTOs (Data Transfer Objects) for method parameters.

### 4. API Routes (`src/app/api/`)
- RESTful endpoints that delegate to services
- Input validation (file size, type, format)
- Consistent response format: `{ success, data, error }`
- Error logging with context

**Pattern**: Middleware-like approach for validation, then service call, then response.

### 5. Components (`src/components/`)
- Client-side forms (`'use client'` directive)
- State management with `useState` hooks
- Form validation before submission
- Toast/alert notifications for feedback

**Pattern**: Pass callbacks (`onSubmit`, `onError`) from parent pages.

## Core Workflows

### Workflow 1: Create & Upload Lab Record

```
1. Page calls LabRecordService.createLabRecord({ patientId, caseId, patientName })
   → Creates document with empty departments
   
2. User clicks LabUploadForm
   → Selects department + field + file
   → Calls POST /api/lab-record/upload
   
3. API route:
   a) Validate file (size, type)
   b) Upload to Cloudinary with upload_stream
   c) Update nested field: departments.${dept}.${field}.fileUrl = cloudinaryUrl
   d) Track file in uploadedFiles array
   e) Return updated document
```

### Workflow 2: Approve/Reject Lab Record

```
1. Consultant views lab record
2. Reviews all department fields
3. Calls LabRecordService.markAsApproved(recordId, consultantId, notes)
   → Sets status = "approved", approvedAt = now, approvedBy = consultantId
4. Case status updates to "approved"
5. DMS initiates certificate generation
```

## File Reference Guide

| File | Purpose | Key Functions |
|------|---------|---|
| `src/models/LabRecord.ts` | Mongoose schema | `updateDepartmentField()`, `addUploadedFile()`, `markAsApproved()` |
| `src/lib/mongooseConnect.ts` | DB connection | `connectDB()` |
| `src/lib/cloudinary.ts` | File upload | `uploadToCloudinary()`, `deleteFromCloudinary()` |
| `src/app/api/lab-record/upload/route.ts` | Upload endpoint | `POST /api/lab-record/upload` |
| `src/components/LabUploadForm.tsx` | Upload form UI | Form state, validation, API call |
| `src/lib/services/LabRecordService.ts` | Business logic | `createLabRecord()`, `updateDepartmentField()`, `markAsApproved()` |

## Common Patterns

### Pattern 1: Validate ObjectId
```typescript
if (!mongoose.Types.ObjectId.isValid(userId)) {
  throw new Error('Invalid user ID format');
}
```

### Pattern 2: Update Nested Field
```typescript
const updatePath = `departments.${department}.${fieldName}`;
await LabRecordModel.findByIdAndUpdate(
  id,
  { $set: { [updatePath]: value } },
  { new: true }
);
```

### Pattern 3: Fetch with Populate
```typescript
const record = await LabRecordModel.findById(id)
  .populate('patientId', 'name email')
  .populate('caseId', 'caseId status');
```

### Pattern 4: API Response
```typescript
return NextResponse.json({
  success: true,
  data: savedRecord.toObject()
}, { status: 200 });
```

## Department Field Mapping

Reference when implementing file uploads or field updates:

**Medicine** (15 fields):
- Tests with value+fileUrl: HBS, HCV, HIV, VDRL, TB, DM
- Tests with fileUrl only: ChestXray, ECG
- Vitals (numbers): Height, PulseRate, RespirationRate, Temperature
- Strings: BloodPressure, BloodGroup, InfectiousDiseaseScreening

**Ophthalmology** (3 fields):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nooraizafshan/MediServe_V00001](https://github.com/nooraizafshan/MediServe_V00001) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
