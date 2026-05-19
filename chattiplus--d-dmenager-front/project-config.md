---
trigger: always_on
description: Questo file descrive:
---

# DD MANAGER – PROJECT AGENT & TASK LIST

Questo file descrive:
- le REGOLE generali che l’agent deve seguire sul progetto DD Manager;
- le TASK backend (0–13) già eseguite o da usare come riferimento;
- le TASK frontend (FE-0…FE-5) per la web app di gestione.

Usalo come “istruzioni di base” per l’agent (system / project prompt).  
Le task già fatte servono per mantenere coerenza, non per riscrivere tutto da zero.

------------------------------------------------------------
0. LINEE GUIDA GLOBALI PER L’AGENT
------------------------------------------------------------

0.1 Tecnologie e versione
- Backend:
  - Java 17
  - Spring Boot 3.x
  - Maven
  - PostgreSQL
- Frontend:
  - Vite + Vue 3 + TypeScript
  - Vue Router
  - Pinia
  - Axios

0.2 Struttura progetto backend
- Root package: `it.univ.ddmanager`.
- Organizzazione per “feature / modulo”:
  - `user` (model, dto, repository, service, controller, exception)
  - `world` (model, dto, repository, service, controller)
  - `campaign` (model, dto, repository, service, controller)
  - `session` (model, dto, repository, service, controller)
  - `npc`
  - `location`
  - `item`
  - `sessionlog` (timeline / session events)
  - `security` (config, UserPrincipal, CurrentUserService)
  - `common` / `error` (ApiError, eccezioni applicative, RestExceptionHandler)
  - `config` (DataInitializer, ecc.)
  - `controller` generici (health)
- Per ogni modulo:
  - Entity JPA in `*.model`.
  - DTO in `*.dto`.
  - Repository in `*.repository`.
  - Service in `*.service`.
  - Controller REST in `*.controller` o `*.web`.

0.3 Dipendenze e import
- Usare Jakarta:
  - `jakarta.persistence.*` per JPA.
  - `jakarta.validation.*` per validation.
- Usare Lombok per boilerplate (`@Getter`, `@Setter`, `@Builder`, ecc.).
- Usare `org.springframework.security.*`, `org.springframework.web.*`, `org.springframework.data.jpa.*`.

0.4 Convenzioni REST
- Base path API: `/api/...`.
- Convenzioni HTTP:
  - `GET` → 200 OK, ritorna DTO.
  - `POST` → 201 Created, ritorna DTO o location.
  - `PUT` → 200 OK, ritorna DTO aggiornato.
  - `DELETE` → 204 No Content.
- Mai esporre direttamente entity JPA dai controller: usare sempre DTO di risposta.

0.5 Sicurezza e ruoli
- Ruoli: `ROLE_ADMIN`, `ROLE_GM`, `ROLE_PLAYER`, `ROLE_VIEWER`.
- Autenticazione: HTTP Basic (email + password BCryt).
- Regole generali:
  - `ADMIN` / `GM`: possono creare/modificare/eliminare domini di gioco (world, campaign, session, npc, location, item, session events).
  - `PLAYER` / `VIEWER`: solo lettura, con filtri su visibilità (`isVisibleToPlayers`).
- Endpoint pubblici:
  - `POST /api/auth/register`
  - `POST /api/auth/login`
- Tutti gli altri endpoint → autenticazione obbligatoria.
- Si possono usare:
  - `@PreAuthorize("hasAnyRole('ADMIN','GM')")` per mutate,
  - oppure regole in `SecurityFilterChain` con `.requestMatchers()`.

0.6 Gestione errori
- Struttura di errore standard: `ApiError` con almeno:
  - `timestamp`
  - `status`
  - `error`
  - `message`
  - `path`
- Mappatura eccezioni tipiche:
  - `EmailAlreadyUsedException` → 409
  - `InvalidRoleException` → 400
  - `InvalidCredentialsException` → 401
  - `ResourceNotFoundException` → 404
  - `AccessDeniedException` / ruoli non validi → 403
  - tutti gli altri errori inattesi → 500
- Usare un `@RestControllerAdvice` globale per mappare eccezioni in `ApiError`.

0.7 Testing backend
- Usare JUnit 5, Spring Boot Test, MockMvc.
- Pattern tipico:
  - `@SpringBootTest`
  - `@AutoConfigureMockMvc`
- Usare `httpBasic()` di `spring-security-test` per simulare autenticazione.
- Per ogni macro-modulo:
  - almeno 1–2 test “happy path”,
  - almeno 1–2 test di permessi (401/403),
  - almeno 1–2 test “not found” / validazioni.

0.8 Frontend
- SPA Vue 3 (Vite + TS).
- Stato globale:
  - store `auth` in Pinia con:
    - `email`, `password` (solo per dev), `nickname`, `roles`, `isAuthenticated`.
- Axios:
  - client centrale con `baseURL` da `VITE_API_BASE_URL`.
  - interceptor che aggiunge header `Authorization: Basic base64(email:password)` se l’utente è loggato.
- Routing minimo:
  - `/login`
  - `/worlds`
  - `/worlds/:id`
  - `/campaigns/:id`
  - `/sessions/:id` (timeline)

------------------------------------------------------------
1. BACKEND TASK LIST (0–13)
------------------------------------------------------------

Le Task 0–11 risultano già implementate nel progetto corrente (vedi `devlog.md`), ma restano come riferimento.  
Le Task 12–13 sono per documentazione e Postman.

---------------------------------
Task 0 – Setup iniziale (backend)
---------------------------------
- Creare progetto Maven Spring Boot 3 (Java 17) con dipendenze:
  - web, data-jpa, security, validation, Lombok, PostgreSQL driver.
- Configurare `application.yml`:
  - datasource Postgres locale (`dd_manager`),
  - `spring.jpa.hibernate.ddl-auto=update` (solo dev),
  - logging SQL formattato.
- Creare `DdManagerApplication` in `it.univ.ddmanager`.
- Aggiungere entità `Role` e `User` minime con repository:
  - `Role(name)`
  - `User(email, password, nickname, Set<Role>)`
- `DataInitializer` per creare i ruoli base:
  - `ROLE_ADMIN`, `ROLE_GM`, `ROLE_PLAYER`, `ROLE_VIEWER`.

----------------------------
Task 1 – Health check REST
----------------------------
- Aggiungere `HealthController` con:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chattiplus/D-Dmenager-Front](https://github.com/chattiplus/D-Dmenager-Front) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
