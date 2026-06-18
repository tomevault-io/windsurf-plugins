---
trigger: always_on
description: - NIKAD ne implementiraj autorizaciju samo na klijentskoj strani (JS routing/UI skrivanje).
---

# Cursor AI Rules of Engagement — Vibe-Audit Framework v3.0
# Primenjuje se automatski za sve Cursor, Cline, Copilot i kompatibilne AI asistente.
# Referentni standardi: OWASP Top 10:2025 | OWASP API Security Top 10:2023 |
#   WCAG 2.2 AA | GDPR Čl.25/30/32 | SonarQube "Sonar way for AI Code" (2025) |
#   OWASP LLM Top 10:2025 | OWASP Agentic Top 10:2026 | Trivy 0.50+ | Bandit AST | ASVS 5.0

---

## 1. SIGURNOST I GRANICE POVJERENJA (OWASP Top 10:2025)

### A01:2025 — Broken Access Control
- NIKAD ne implementiraj autorizaciju samo na klijentskoj strani (JS routing/UI skrivanje).
- Svaki server-side endpoint mora imati eksplicitnu proveru permisija (RBAC/ABAC).
- Proveri IDOR: pristup resursima mora biti validovan prema identity-u korisnika koji šalje zahtjev, ne samo prema ID-u resursa.
- SSRF zaštita: nikad ne prosljeđuj korisnički unos kao URL za interni HTTP poziv bez whitelist provere.

### A02:2025 — Security Misconfiguration
- Ne ostavljaj debug mode, verbose error poruke ili detailed stek traces u produkcijskom kodu.
- Postavi security headere: `Content-Security-Policy`, `X-Frame-Options`, `X-Content-Type-Options`, `Strict-Transport-Security`.
- CORS: nikad `Access-Control-Allow-Origin: *` na authenticated endpointima.
- Promijeni sve default kredencijale i podrazumjevane konfiguracije.

### A03:2025 — Software Supply Chain Failures
- **ZABRANA haluciniranih paketa**: Pre importa nove biblioteke, proveri da li postoji u `package.json`, `pyproject.toml`, `requirements.txt` ili ekvivalentnom manifest fajlu.
- Propinkuj verzije zavisnosti (pin, ne `^latest`).
- Ne instaliraj pakete bez eksplicitnog odobrenja korisnika.
- Generiraj SBOM za svaki release (preporučeno: CycloneDX ili SPDX format).

### A04:2025 — Cryptographic Failures
- Nikad ne koristiti MD5 ili SHA-1 za hashovanje lozinki. Koristiti bcrypt, Argon2id ili scrypt.
- Tajne u tranzitu: obavezno TLS 1.2+. Nikad plain HTTP za auth endpointe.
- Enkripcija at-rest za sve PII (Personally Identifiable Information).

### A05:2025 — Injection
- **SQL**: UVIJEK koristi parametrizovane upite ili ORM. Nikad string concatenation.
- **Shell**: koristi array-based subprocess pozive (ne `shell=True` u Pythonu, ne `exec(userInput)` u JS).
- **XSS**: escapuj sve korisničke unose koji se renderuju u HTML-u. Koristi template engine sa auto-escapingom.
- **Template Injection**: nikad ne prosljeđuj korisnički unos direktno u template engine.

### A07:2025 — Authentication Failures
- JWT: proveri `exp`, `iss`, `aud`. Nikad ne prihvataj JWT bez provere potpisa.
- Session management: HttpOnly, Secure, SameSite=Strict za cookies.
- Rate limiting na auth endpointima (brute force zaštita).

### A08:2025 — Software or Data Integrity Failures
- **ZABRANA eval()** i `Function()` konstruktora sa korisničkim unosom.
- Nikad ne deserijalizuj nepovjerljive podatke bez provere sheme.
- Base64 dekodiranje korisničkog unosa: uvijek validiraj sadržaj nakon dekodiranja.

### A09:2025 — Security Logging & Alerting Failures
- Loguj sve auth događaje (login, logout, failed attempts, privilege escalation).
- Nikad ne loguj tajne, lozinke, tokene ili PII u plaintext logove.
- Implementiraj alert za abnormalne obrasca (npr. 100+ failed logins u 1 min).

### A10:2025 — Mishandling of Exceptional Conditions
- **ZABRANA praznih catch blokova**: svaki `except/catch` mora ili logirati grešku (structured log sa context-om) ili rethrowati.
- Ne vraćaj generičke 500 greške bez logiranja detalja interno.
- Implementiraj graceful degradation: ako externa zavisnost pada, vrati jasnu grešku, ne crash.

---

## 2. API SIGURNOST (OWASP API Security Top 10:2023)

Kada implementiraš API endpoint:
- **BOLA (API1)**: proveri da li korisnik ima pravo na tačno taj objekat (ne samo da li je autentifikovan).
- **BFLA (API5)**: admin funkcije moraju biti zaštićene po ulozi, ne samo po autentifikaciji.
- **Unrestricted Resource Consumption (API4)**: implementiraj rate limiting i paginaciju.
- **SSRF (API7)**: whitelist dozvoljena odredišta za server-side HTTP pozive.
- **Improper Inventory Management (API9)**: dokumentuj sve endpointe u OpenAPI specifikaciji. Nikad ne ostavljaj zastarjele endpointe aktivne bez deprecation oznake.

---

## 3. ZAVISNOSTI I LANAC SNABDEVANJA

- Proveri sve pakete u registrima (npm, PyPI) pre instalacije — ne instaliraj halucinovane pakete.
- Koristiti `trivy fs --scanners vuln,misconfig,secret,license .` periodično za skeniranje.
- Koristiti `bandit -r . -f json` za Python projekte u CI/CD-u.
- Nikad ne ubacivati pakete sa jednim maintainerom ili bez commit-a u zadnjih 12 mj bez provere.

---

## 4. TAJNE I KREDENCIJALI

- **NIKAD** hardkodiraj API ključeve, lozinke, DB connection strings, tokene ili sertifikate direktno u kod.
- Koristi environment varijable učitane iz sigurnog sistema (vault, GitHub Secrets, GitLab CI/CD Variables, AWS Secrets Manager, Azure Key Vault).
- Implementiraj startup validaciju: aplikacija treba da odbije pokretanje ako nedostaje obavezna env var (fail-fast princip).
- Dodaj `.gitignore` pravila za sve `.env*` fajlove.
- Dodaj secret scanning u CI/CD pipeline (Trivy secret scan, GitHub secret scanning).

---

## 5. POUZDANOST I RUKOVANJE GREŠKAMA

- Svaka async operacija mora imati timeout definisan.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zoxknez/post-vibe-audit](https://github.com/zoxknez/post-vibe-audit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->
