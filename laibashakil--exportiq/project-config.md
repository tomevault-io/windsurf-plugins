---
trigger: always_on
description: Generates 3-5 prioritized actions with deadlines
---

# ExportIQ — Pakistan Textile Export Compliance Agent
### AISeekho 2026 Google Antigravity Hackathon — Challenge 1

---

## Project Overview

An agentic AI system that ingests EU/UK compliance regulations, factory audit reports, and export data to automatically detect compliance gaps, calculate financial risk in PKR, generate prioritized action chains, and simulate execution — protecting Pakistani textile factories from losing billions in export orders.

**Hackathon:** AISeekho 2026 Google Antigravity Hackathon
**Challenge:** Challenge 1 — Autonomous Content-to-Action Agent
**Submission Deadline:** May 20, 2026
**Team Size:** Small team

---

## Tech Stack

| Layer | Technology | Why |
|---|---|---|
| Mobile App | Expo (React Native) | Already familiar, Expo Go QR scan sufficient for demo |
| Backend | Python 3.11 + FastAPI | Fast to build, async-friendly for agent pipelines |
| Agent Orchestration | Google Antigravity + LangGraph | Antigravity is mandatory; LangGraph manages multi-agent state |
| LLM | Gemini 2.5 Pro (Vertex AI) | Required for Antigravity; best PDF/doc understanding |
| Database | Firebase Firestore | Real-time updates to mobile app, no server config |
| File Storage | Firebase Storage | Storing uploaded PDFs and generated documents |
| Hosting | Google Cloud Run | Free credits from hackathon, serverless, zero config |
| PDF Parsing | PyMuPDF (fitz) + Gemini | Extract text from audit reports and regulation PDFs |
| Auth | Firebase Auth (anonymous) | Quick demo auth, no signup friction |

---

## Project Structure

```
ExportIQ/
├── CLAUDE.md                          # This file — AI coding assistant instructions
├── README.md                          # Hackathon submission README
│
├── backend/                           # FastAPI backend on Google Cloud Run
│   ├── main.py                        # FastAPI app entry point
│   ├── requirements.txt
│   ├── Dockerfile
│   ├── .env.example
│   │
│   ├── agents/                        # All 6 LangGraph agents
│   │   ├── __init__.py
│   │   ├── orchestrator.py            # Master agent — coordinates all 6
│   │   ├── regulation_agent.py        # Agent 1: Parse EU/UK regulation PDFs
│   │   ├── factory_profile_agent.py   # Agent 2: Parse factory audit CSVs/PDFs
│   │   ├── gap_detection_agent.py     # Agent 3: Cross-reference & find gaps
│   │   ├── financial_impact_agent.py  # Agent 4: Calculate PKR risk exposure
│   │   ├── action_chain_agent.py      # Agent 5: Generate 3-5 prioritized actions
│   │   └── execution_agent.py         # Agent 6: Simulate action execution
│   │
│   ├── tools/                         # Agent tools (Antigravity Skills equivalent)
│   │   ├── pdf_parser.py              # PyMuPDF + Gemini PDF extraction
│   │   ├── csv_processor.py           # Factory data CSV ingestion
│   │   ├── contradiction_detector.py  # Cross-source conflict detection
│   │   ├── document_generator.py      # Generate buyer emails, checklists, reports
│   │   ├── compliance_scorer.py       # 0-100 compliance scoring logic
│   │   └── firestore_client.py        # Firebase read/write helpers
│   │
│   ├── models/                        # Pydantic data models
│   │   ├── factory.py
│   │   ├── regulation.py
│   │   ├── gap_report.py
│   │   └── action_chain.py
│   │
│   ├── api/                           # FastAPI route handlers
│   │   ├── upload.py                  # POST /upload — ingest PDFs/CSVs
│   │   ├── analyze.py                 # POST /analyze — trigger full agent pipeline
│   │   ├── actions.py                 # GET /actions/{factory_id} — get action chain
│   │   ├── simulate.py                # POST /simulate — run execution simulation
│   │   └── status.py                  # GET /status/{job_id} — poll agent progress
│   │
│   └── mock_data/                     # Mock inputs for demo
│       ├── factories/
│       │   ├── fwi_audit.pdf      # Mock factory audit report
│       │   ├── cfw_audit.pdf
│       │   └── factory_export_data.csv       # Pakistan-Textile-Mills-Council-style export volumes
│       └── regulations/
│           ├── eu_cbam_rules.pdf             # Real EU CBAM PDF (publicly available)
│           ├── uk_modern_slavery_act.pdf
│           └── eu_supply_chain_directive.pdf
│
├── mobile/                            # Expo React Native app
│   ├── app.json
│   ├── package.json
│   ├── App.js
│   │
│   ├── screens/
│   │   ├── SplashScreen.js             # First-launch splash
│   │   ├── HomeScreen.js               # Compliance score dashboard + deadlines badge
│   │   ├── ComplianceScreen.js         # Per-regulation breakdown (Status tab)
│   │   ├── ActionCenterScreen.js       # Agent-generated action items (Fix It tab)
│   │   ├── DocumentVaultScreen.js      # Upload PDFs, view generated docs (Documents tab)
│   │   ├── AgentTraceScreen.js         # Live Antigravity reasoning trace log (dev route)
│   │   ├── UploadScreen.js             # PDF upload + auto-trigger analysis
│   │   ├── AnalysisProgressScreen.js   # Animated progress while pipeline runs
│   │   ├── HowItWorksScreen.js         # Scoring + compliance explainer
│   │   ├── EditEmailScreen.js          # Compose/edit a generated buyer email
│   │   ├── SettingsScreen.js           # User preferences + cross-factory export

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [laibashakil/ExportIQ](https://github.com/laibashakil/ExportIQ) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
