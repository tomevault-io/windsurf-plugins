---
trigger: always_on
description: As an expert AI pair programmer, my goal is to deliver precise, high-quality code modifications by operating as an autonomous agent. I will follow your instructions meticulously, continuing to work through my plan until the request is fully resolved.
---

# Copilot instructions

### **Core Directive**
As an expert AI pair programmer, my goal is to deliver precise, high-quality code modifications by operating as an autonomous agent. I will follow your instructions meticulously, continuing to work through my plan until the request is fully resolved.

### **Core Principles**
1. **Minimize Scope of Change**  
   - Identify the smallest unit (function, class, or module) that fulfills the requirement.  
   - Do not modify unrelated code.  
   - Avoid refactoring unless required for correctness or explicitly requested.

2. **Preserve System Behavior**  
   - Ensure the change does not affect existing features or alter outputs outside the intended scope.  
   - Maintain original patterns, APIs, and architectural structure unless otherwise instructed.

3. **Graduated Change Strategy**  
   - **Default:** Implement the minimal, focused change.  
   - **If Needed:** Apply small, local refactorings (e.g., rename a variable, extract a function).  
   - **Only if Explicitly Requested:** Perform broad restructuring across files or modules.

4. **Clarify Before Acting on Ambiguity**  
   - If the task scope is unclear or may impact multiple components, stop and request clarification.  
   - Never assume broader intent beyond the described requirement.
  
### **Detailed Overiew of the Codebase**
Overview
- Django backend API/service in Python using Django REST Framework and pandas/SQLAlchemy for data processing.
- Purpose: ERP-style ingestion from external systems (IKEA portal, GST portal, E-Invoice portal), normalize into internal models, generate GST filings (workings + JSON), and manage e-invoice lifecycle.
- Key external clients (custom/classes.py):
  - IkeaDownloader/BaseIkea: fetches reports, sales registers, inventory, statements, PDFs, etc.
  - Gst: GST portal session + JSON/ZIP download, invoice fetch, EINVOICE data access, JSON generation.
  - Einvoice: NIC e-invoice portal session, bulk upload, recent IRN retrieval.
- Auth/session for external systems is stored per Django user in app.company_models.UserSession (username/password/cookies/config). All client sessions load and persist cookies through this model.

Project structure (what matters)
- app/api.py: Primary REST endpoints (function-based views). Central place for GST/E-Invoice actions, Excel/PDF responses, and session updates.
- custom/Session.py: Base HTTP session with logging, cookie persistence, error handling, and enforced base_url. Subclasses set key, base_url, is_logged_in(), login(). Responses are HTML-logged to logs/<key>.html.
- custom/classes.py: Concrete clients and workflows.
  - BaseIkea/IkeaDownloader: report() helpers via curl templates, login flow, many report/download utilities (sales_reg, gstr_report, damage_proposals, pending statements, eway, einvoice_json, etc.).
  - Gst: portal automation (captcha login, downloads, JSON building, EINVOICE data).
  - Einvoice: NIC portal automation (captcha/login, bulk upload, IRN fetch).
- app/report_models.py: Report model pattern.
  - Define <Something>Report models with inner Report that declares fetcher (IkeaDownloader method), column_map, preprocessing, caching options.
  - BaseReportModel handles saving DataFrame to DB and update_db() orchestration.
  - DateReportModel and EmptyReportModel encapsulate date-ranged and full refresh behaviors.
  - Concrete reports: SalesRegisterReport, IkeaGSTR1Report, DmgShtReport, StockHsnRateReport, PartyReport, GSTR1Portal.
- app/erp_import.py: Import pipeline from external reports to ERP models.
  - SalesImport, MarketReturnImport, StockImport, PartyImport.
  - GstFilingImport orchestrates report refresh (in parallel) then transactional imports; applies SalesChanges deltas.
- app/erp_models.py: Core ERP models with composite primary keys and cross-table relations via from_fields/to_fields.
  - Sales, Purchase, Inventory, Stock, Party, SalesChanges, Discount, StockAdjustment.
  - Sales.user_objects manager scopes queries to the authenticated user’s companies.
- app/gst.py: GST monthly return generator.
  - Builds workings Excel, matches internal vs portal, computes zero-rate logic, constructs GST JSON, pulls extra HSN from e-invoice if needed.
- app/einvoice.py: E-invoice JSON builder from internal Sales and Inventory; date tweaks and JSON encoding.
- app/management/commands/monthly_gst.py: CLI command to run monthly GST import/marking by period.

Core concepts and flows
- Session + login
  - Each external client Session subclass uses app.company_models.UserSession keyed by (user, key) to load username/password/cookies/config.
  - check_login decorator (api.py) instantiates the client for request.user and calls is_logged_in(). If false, responds 501 with {"key": client.key}. Frontend should then:
    - POST /get_captcha with key -> returns image.
    - POST /captcha_login with key, captcha -> stores cookies via client.login() and UserSession.update_cookies().
- Data ingestion pipeline
  - Reports fetched via IkeaDownloader.* methods are mapped in Report inner classes (fetcher attribute).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Venkatesh2304/myerpv3](https://github.com/Venkatesh2304/myerpv3) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
