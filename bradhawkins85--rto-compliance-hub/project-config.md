---
trigger: always_on
description: Design and implement a comprehensive, modular compliance management system for Registered Training Organizations (RTOs). The system centralizes governance, training, feedback, compliance tracking, and resource management into one platform.
---

# Compliance Management and Governance Platform for RTOs

## Objective
Design and implement a comprehensive, modular compliance management system for Registered Training Organizations (RTOs). The system centralizes governance, training, feedback, compliance tracking, and resource management into one platform.

---

## 1. Core Modules & Functional Requirements

### 1.1 Governance
- Manage and store policies, procedures, and compliance documentation.
- Track financial compliance, child safety, leadership fit and proper tests, workforce adequacy, and insurance documentation.
- Support continuous improvement workflows with version control and audit history.
- Role-based permissions for editing and reviewing policies.
- Integration with Google Drive for document storage and linking.

### 1.2 Training Product Management
- Link training products to associated SOPs, validation reports, and instructional materials.
- Map assessment strategies and schedules for each training product.
- Support both accredited and non-accredited training.
- AI analysis to identify misalignments or outdated materials.

### 1.3 Feedback Management
- **Learner Feedback:** Collect via surveys (JotForm) by trainer, course, or learner type.
- **Employer Feedback:** Capture outcomes-based metrics and anecdotal input.
- **Industry Feedback:** Gather surveys and audio anecdotes from stakeholders (employers, safety officers, training coordinators).
- Integrate with AI sentiment and trend analysis for feedback aggregation.
- Export summaries and generate compliance insights.

### 1.4 Professional Development
- Manage staff PD cycles from planning to completion.
- Track vocational and industry currency, including work placements and skill maintenance.
- Store credentials, certificates, and expiration reminders.
- Link PD activities to compliance standards and staff records.

### 1.5 Resource Management
- **Assets:** Track cranes, plant, tablets, laptops, lifting equipment, and maintenance schedules.
- **Infrastructure:** Manage classrooms, offices, exclusion zones, and training yards.
- **Learning Resources:** Central library of guides, videos, and podcasts linked to training modules.
- **Systems Integration:** Connect to third-party systems (SMS, LMS, CRM, Xero).

### 1.6 Complaints & Appeals
- Workflow-driven process for complaint logging, tracking, and resolution.
- Capture student demographics and incident details.
- Link complaint outcomes to policy and continuous improvement actions.
- Automated reporting to meet compliance obligations.

### 1.7 Compliance Tracking & Mapping
- Compliance map linking all 29+ RTO standards to related policies, procedures, training, and evidence.
- Dashboard to visualize current compliance status.
- Automated reminders for renewals, audits, or missing documentation.

### 1.8 HR & Onboarding
- Centralized staff database syncing with Xero for payroll and positions.
- Onboarding workflows integrated with compliance documents and PD requirements.
- Department-level access control (training, admin, management).
- API connections to Accelerate for trainer and student details.

---

## 2. Data Architecture & Integrations
- **Database:** Relational (PostgreSQL / Airtable equivalent).
  - Tables: Staff, Training, Policies, Assets, Feedback, Standards, Complaints.
- **APIs & Integrations:**
  - JotForm (webhooks for data ingestion).
  - Airtable or internal SQL backend.
  - Accelerate API for student/trainer data.
  - Xero API for payroll sync.
  - n8n for automation workflows.
  - Grafana for dashboards.
  - Google Drive API for document links and storage.

---

## 3. Automation & AI Features
- **AI Analysis:** Process student and employer feedback to detect trends and improvement opportunities.
- **Auto-Reminders:** Schedule compliance renewals, PD expirations, and document reviews.
- **Smart Workflows:** Conditional logic for forms (e.g., training type triggers relevant SOP).
- **Predictive Insights:** Forecast compliance gaps or risks based on feedback and training trends.

---

## 4. User Experience
- Portal interface for staff with personalized dashboards.
- Dynamic navigation by role (trainer, admin, leadership).
- Search and filter across all modules (resources, policies, training).
- Embedded JotForm and internal forms for feedback and submissions.
- Consistent branding and responsive design for desktop and mobile.

---

## 5. Security & Permissions
- Role-based access control (RBAC) for users and departments.
- Secure API authentication (OAuth2 / JWT).
- Audit logs for all document changes and user actions.
- Encrypted data storage for PII and compliance evidence.

---

## 6. Future Features (Scalable Roadmap)
- AI compliance auditor for proactive alerts.
- Predictive staff training recommendations.
- Self-service employer and auditor portals.
- ChatGPT integration for document Q&A (policy search assistant).
- Data visualizations for compliance trends and performance metrics.

---

## 7. Technical Stack Recommendations
- **Frontend:** React (Next.js) + TailwindCSS
- **Backend:** Node.js / FastAPI

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bradhawkins85) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
