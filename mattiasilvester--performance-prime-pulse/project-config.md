---
trigger: always_on
description: **SE INCONTRI CONFLITTI, PROBLEMI O ALTRO FERMATI E DIMMELO - NON CONTINUARE**
---

# PERFORMANCE PRIME PULSE - REGOLE DI SVILUPPO
# 3 Settembre 2025 - PROGETTO IN SVILUPPO ATTIVO

## ⚠️ REGOLA CRITICA: CONFLITTI E PROBLEMI
**SE INCONTRI CONFLITTI, PROBLEMI O ALTRO FERMATI E DIMMELO - NON CONTINUARE**

- Se trovi conflitti con codice esistente → FERMATI
- Se trovi problemi di import o dipendenze → FERMATI  
- Se trovi errori TypeScript non risolti → FERMATI
- Se trovi incompatibilità con pattern esistenti → FERMATI
- **NON procedere** senza aver segnalato e risolto il problema
- Se un prompt richiede modifiche già esistenti, fermati, avvisa l'utente e non duplicare il lavoro

## 🛡️ REGOLA ZERO BREAKING: FIX E FEATURE NON DEVONO ROMPERE NULLA
**Tutti i fix e le modifiche NON devono rompere nulla. Qualora qualcosa potesse creare problemi a una funzione o a una feature → FERMATI E DIMMELO.**

- Vale per **ogni fix**, **nuove feature**, **refactoring** e **qualsiasi modifica** al codice
- Prima di applicare un fix: valuta se può impattare funzionalità esistenti o flussi utente
- Se un fix/feature potrebbe creare problemi (es. regex che cambiano comportamento, tipi che rompono chiamate, logica che altera risultati) → **FERMATI**, segnala il rischio e chiedi conferma
- **NON** procedere con modifiche che potrebbero rompere funzioni o feature senza averlo segnalato

## 🔒 BLOCCO FUNZIONALITÀ — NON TOCCARE (11 Febbraio 2026)
**Prima di implementare qualsiasi nuova feature: queste funzionalità sono TESTATE E FUNZIONANTI. Se una modifica rischia di impattare anche solo UNA, FERMATI e chiedi prima.**

Riferimento completo e aggiornabile: **docs/BLOCCO_FUNZIONALITA.md** (aggiornare a fine sessione se serve).

**Funzionalità bloccate (non modificare):** Autenticazione (login, logout, reset password). Dashboard (OverviewPage KPI reali, prossimi appuntamenti, stato vuoto). Agenda/Calendario (vista giorno/settimana, drag&drop, blocco slot). Prenotazioni (lista filtrabile, conferma/cancella/completa, conteggio card). Clienti (lista, aggiungi, dettaglio). Servizi e Tariffe (CRUD). Profilo (visualizzazione e modifica). Costi e Spese (gestionale). Report e Analytics (Andamento, export PDF Analytics/Commercialista con e senza dati, Report Settimanale; import jspdf-autotable DEVE restare: `import 'jspdf-autotable'` + `(doc as any).autoTable({...})`). Recensioni. Abbonamento (trial, badge). Notifiche e Promemoria (push, promemoria programmati). Onboarding (tour 13 step, "Rivedi guida"). SuperAdmin (dashboard, CORS). Landing. Email (benvenuto, reminder trial). Safari iOS (input, select, modal). Vercel Analytics (inject in main).

**Regole:** (1) Nuove feature ADDITIVE. (2) Modifiche solo alle righe necessarie. (3) Dopo ogni modifica: pnpm build:pro → 0 errori. (4) Non cambiare import 'jspdf-autotable'. (5) I test manuali che passavano devono continuare a passare.

## 📚 RIFERIMENTO PROBLEMI COMUNI
**PRIMA DI RISOLVERE UN PROBLEMA, CONSULTA:** `PROBLEMI_COMUNI_E_SOLUZIONI.md`

Questo documento contiene:
- 23 problemi comuni riscontrati nel progetto con soluzioni testate
- Pattern riutilizzabili per problemi simili
- File coinvolti e esempi di codice corretti/errati
- Regole generali per prevenire problemi futuri

**Quando incontri un problema:**
1. Cerca nel documento `PROBLEMI_COMUNI_E_SOLUZIONI.md` se esiste già una soluzione
2. Se esiste, applica la soluzione documentata
3. Se non esiste, analizza il problema e documenta la soluzione nel documento

## ⚠️ CRITICAL VITE COMPATIBILITY RULE
- This project uses Vite and is NOT compatible with styled-jsx
- NEVER use <style jsx> or styled-jsx syntax
- NEVER suggest styled-jsx solutions
- Use ONLY:
  * Inline styles with style={{}}
  * Tailwind CSS classes
  * Separate .css files
  * CSS modules (.module.css)

## 🎯 **STATO ATTUALE: PRONTO PER PRODUZIONE CON PRIMEBOT E FOOTER GLASS EFFECT 🚀**

**Performance Prime Pulse** è un'applicazione React completa e pronta per la produzione con sistema di autenticazione completo, gestione errori avanzata, landing page ottimizzata, sistema filtri interattivi, overlay GIF esercizi, automazione feedback 15 giorni, PrimeBot OpenAI completamente integrato con risposte intelligenti e formattazione markdown, footer con effetto vetro identico all'header, e configurazione completa per deploy. Ultimi sviluppi: 16 Gennaio 2025 - Sessione 12.

---

## 🚀 **ARCHITETTURA IMPLEMENTATA**

### **Flusso Utente Completo**
1. **Landing Page** (`/`) - Presentazione prodotto
2. **Registrazione** (`/auth/register`) - Creazione account
3. **Login** (`/auth/login`) - Accesso utente
4. **Dashboard** (`/dashboard`) - App principale protetta
5. **Logout** - Ritorno alla landing

### **Componenti Principali**
- `App.tsx` - Routing e gestione sessione
- `LandingPage.tsx` - Landing page completa
- `LoginPage.tsx` - Autenticazione utente
- `RegisterPage.tsx` - Registrazione utente
- `Dashboard.tsx` - App principale con logout
- `ProtectedRoute.tsx` - Protezione route autenticate

### **Componenti SuperAdmin (LOCKED)**
- `src/lib/adminApi.ts`
- `src/components/admin/UserManagementTable.tsx`
- `src/pages/admin/AdminUsers.tsx`
- `supabase/functions/admin-users/index.ts`
- `supabase/functions/_shared/cors.ts`

---

## 🔧 **STEP COMPLETATI CON SUCCESSO**

### **STEP 1: FIX ARCHITETTURA LANDING → AUTH → APP** ✅
- Routing completo implementato

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Mattiasilvester) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
