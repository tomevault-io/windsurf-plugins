---
trigger: always_on
description: Build a simple, free PCOD/PCOS tracking platform for real users.
---

# AGENTS.md - PCOD Tracker Project

## Project Goal

Build a simple, free PCOD/PCOS tracking platform for real users.

The product helps users track:

* Period cycles
* Symptoms
* Weight
* Medications
* Basic analytics
* Doctor-friendly reports

Core principle:

No hidden paywalls. No ads. No locked health history. Core health tracking stays free.

---

## Target Users

Women with PCOD/PCOS who currently track health information across multiple places such as period apps, notes, WhatsApp, memory, or paper.

Initial users:

* Girlfriend
* Her friends with PCOD/PCOS

Success metric:

At least 5 users use the app.
At least 3 users continue using it after 30 days.

---

## Product Philosophy

Build useful before clever.

Do not add AI, community, doctor marketplace, diet plans, or predictions in MVP.

The app should be:

* Simple
* Fast
* Private
* Mobile-friendly
* Easy to log daily
* Useful during doctor visits

---

## MVP Features

### 1. Dashboard

Show:

* Days since last period
* Current cycle length
* Latest weight
* Symptoms logged this week
* Medication adherence percentage

---

### 2. Cycle Tracker

User can:

* Add period start date
* Add period end date
* Select flow: LIGHT, MEDIUM, HEAVY
* View cycle history
* See average cycle length

---

### 3. Symptom Tracker

User can log daily symptoms.

Initial symptoms:

* Acne
* Bloating
* Fatigue
* Mood swings
* Anxiety
* Headache
* Cramps
* Hair fall
* Cravings

Each symptom has:

* Date
* Symptom type
* Severity: 1 to 5

---

### 4. Weight Tracker

User can:

* Add weight
* View weight history
* See weight change over time

---

### 5. Medication Tracker

User can:

* Add medicine name
* Add dosage
* Add reminder time
* Mark medicine as taken or missed
* View adherence percentage

---

### 6. Doctor Report

Generate a simple report for last 30/90/180 days.

Report should include:

* Cycle history
* Average cycle length
* Weight trend
* Most common symptoms
* Medication adherence

PDF export can be added after basic report screen works.

---

## Tech Stack

### Backend

Use:

* Java 17 or Java 21
* Spring Boot 3.x
* Spring Web
* Spring Data JPA
* Spring Security
* JWT authentication
* PostgreSQL
* Flyway
* Maven
* Docker

Backend must serve both website and mobile app.

---

### Web Frontend

Use:

* React
* Vite
* Axios
* React Router
* Recharts or Chart.js

Website must be responsive and mobile-friendly.

---

### Mobile App

Use later:

* React Native
* Same backend APIs
* Android first

Do not start mobile app before web MVP is usable.

---

## Backend Package Structure

Use this structure:

com.pcodtracker

* config
* auth
* user
* cycle
* symptom
* weight
* medication
* dashboard
* report
* common
* exception

Each feature should contain:

* Entity
* Repository
* DTO
* Service
* Controller

Example:

cycle/

* Cycle.java
* CycleRepository.java
* CycleRequest.java
* CycleResponse.java
* CycleService.java
* CycleController.java

---

## Database Tables

### users

* id
* name
* email
* password_hash
* age
* height_cm
* created_at
* updated_at

---

### cycles

* id
* user_id
* start_date
* end_date
* flow
* created_at
* updated_at

---

### symptom_logs

* id
* user_id
* log_date
* symptom_type
* severity
* note
* created_at
* updated_at

---

### weight_logs

* id
* user_id
* log_date
* weight_kg
* created_at
* updated_at

---

### medications

* id
* user_id
* name
* dosage
* reminder_time
* active
* created_at
* updated_at

---

### medication_logs

* id
* medication_id
* user_id
* log_date
* taken
* created_at
* updated_at

---

## API Design

All authenticated APIs must use JWT.

### Auth APIs

POST /api/auth/register

Request:

{
"name": "Riya",
"email": "[riya@example.com](mailto:riya@example.com)",
"password": "password123",
"age": 29,
"heightCm": 160
}

POST /api/auth/login

Request:

{
"email": "[riya@example.com](mailto:riya@example.com)",
"password": "password123"
}

Response:

{
"token": "jwt-token"
}

---

### Cycle APIs

POST /api/cycles

GET /api/cycles

GET /api/cycles/summary

DELETE /api/cycles/{id}

---

### Symptom APIs

POST /api/symptoms

GET /api/symptoms

GET /api/symptoms/summary

DELETE /api/symptoms/{id}

---

### Weight APIs

POST /api/weights

GET /api/weights

GET /api/weights/summary

DELETE /api/weights/{id}

---

### Medication APIs

POST /api/medications

GET /api/medications

PATCH /api/medications/{id}

POST /api/medications/{id}/logs

GET /api/medications/adherence

---

### Dashboard API

GET /api/dashboard

Response should combine:

* daysSinceLastPeriod
* currentCycleLength
* latestWeight
* weightChangeLast30Days
* symptomsLoggedThisWeek
* medicationAdherencePercent

---

### Report API

GET /api/reports/health-summary?range=90

Response should include:

* cycle summary
* symptom summary
* weight summary
* medication summary

---

## Security Rules

* Passwords must be hashed using BCrypt.
* Never store plain text passwords.
* Each user can access only their own data.
* Do not expose internal IDs unnecessarily in public response if avoidable.
* Validate all request bodies.
* Return clean error responses.

---

## Error Response Format

Use this format:

{
"timestamp": "2026-06-11T10:00:00",
"status": 400,
"error": "Bad Request",
"message": "Validation failed",
"path": "/api/cycles",
"errors": [

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AdiChow/pcod_pulse](https://github.com/AdiChow/pcod_pulse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
