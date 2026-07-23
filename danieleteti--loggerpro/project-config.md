---
trigger: always_on
description: **Data ultimo aggiornamento:** 2026-02-08
---

# LoggerPro - Stato della Sessione Claude

**Data ultimo aggiornamento:** 2026-02-08

---

## Sessione Corrente (2026-02-08)

### Fix Issue #101 - Delphi 10.3 Rio Compatibility

**Problema:** LoggerPro 2.0 non compilava su Delphi 10.3 Rio a causa di overload mancanti in `TJSONObject.AddPair`.

**Errore originale:**
```
E2250: There is no overloaded version of 'AddPair' that can be called with these arguments
```

**Causa:** In Delphi 10.3, `AddPair(string, string)` non esiste. Gli overload aggiunti in 10.4+:
- `AddPair(const Str, Val: string)`
- `AddPair(const Str: string; const Val: Integer)`

Delphi 10.3 ha solo:
- `AddPair(const Str: string; const Val: TJSONValue)`
- `AddPair(const Pair: TJSONPair)`

**Soluzione implementata (Proposta 3 - Sempre TJSONString):**

Usare sempre `TJSONString.Create()` per wrappare i valori stringa - compatibile con TUTTE le versioni Delphi:

```delphi
// PRIMA (non compilava su 10.3):
lJSON.AddPair('timestamp', DateToISO8601(ALogItem.TimeStamp));
lJSON.AddPair('tid', ALogItem.ThreadID.ToString);

// DOPO (compatibile 10.0 Seattle → 13 Florence):
lJSON.AddPair('timestamp', TJSONString.Create(DateToISO8601(ALogItem.TimeStamp)));
lJSON.AddPair('tid', TJSONString.Create(ALogItem.ThreadID.ToString));
```

**File modificato:** `LoggerPro.JSONLFileAppender.pas` (linee 168-198)

**Vantaggi:**
- ✅ Zero `{$IFDEF}` - codice lineare e pulito
- ✅ Compatibile Delphi 10.0 Seattle → 13 Florence
- ✅ Stesso comportamento garantito su tutte le versioni
- ✅ Più esplicito e chiaro

**Test:** 109 test unitari passano su Delphi 11.3
**Commit:** Da committare
**Issue:** #101 - DA AGGIORNARE (non chiudere, aggiornare con fix)

---

### Fix Issue #105 - Context Rendering con Curly Braces

**Problema:** Discrepanza tra documentazione e implementazione. La documentazione mostrava `{order_id=12345, amount=99.99}` ma l'output effettivo era `order_id=12345 amount=99.99`.

**Soluzione:** Modificati 3 punti nel codice:
1. `LoggerPro.Renderers.pas:117-158` - `RenderContext()` ora usa separatore `, ` e wrapper `{}`
2. `LoggerPro.pas:1781-1817` - `RenderContextToString()` stesso formato per pre-rendered context
3. `LoggerPro.Renderers.pas:239-244` - Rimosso shortcut `PreRenderedContext` da LogFmt (preserva formato logfmt standard)

**Commit:** `c0ad85f` - Pushato su origin/master
**Test:** 101 test unitari passano
**Issue:** #105 CHIUSA

---

## Lavoro Completato in Sessione Precedente (2025-12-21)

### 1. Fix Thread Safety nel Destroy

Risolto race condition durante la distruzione del logger quando altri thread tentano di accodare log.

**Problema:** Un thread potrebbe passare il check `FEnabled`, poi `Destroy` libera `FLoggerThread` causando access violation.

**Soluzione:** Aggiunto flag `FShuttingDown: Boolean` senza lock (la lettura di un Boolean e' atomica):

```delphi
// In TCustomLogWriter
FShuttingDown: Boolean;

// Constructor
FShuttingDown := False;

// Destructor (prima cosa)
FShuttingDown := True;

// Nei metodi Log e EnqueueLogItem
Assert(not FShuttingDown, 'Cannot log: logger is shutting down');
if FShuttingDown then Exit;
```

| Build | Comportamento |
|-------|---------------|
| Debug | `EAssertionFailed` - rivela il bug immediatamente |
| Release | Silent drop - graceful degradation |

**Test aggiunti** (`ThreadSafetyTestU.pas`):
- `TestDestroyWhileThreadsAreLogging` (x5)
- `TestDestroyWhileThreadsAreLoggingWithContext` (x5)
- `TestRapidCreateDestroy`

---

### 2. Exception Logging con Stack Trace Pluggabile

```delphi
// Senza stack trace formatter
Log.LogException(E);
Log.LogException(E, 'Operation failed');
Log.LogException(E, 'Operation failed', 'MYTAG');

// Con stack trace formatter (JCL, madExcept, EurekaLog, etc.)
Log := LoggerProBuilder
  .WithStackTraceFormatter(
    function(E: Exception): string
    begin
      Result := JclLastExceptStackListToString;
    end)
  .WriteToConsole.Done
  .Build;
```

**Design:**
- `TStackTraceFormatter = TFunc<Exception, string>` - nessuna interfaccia custom
- Zero dipendenze inverse - la libreria di stack trace non dipende da LoggerPro
- TAG sempre come ultimo parametro (consistente con tutta l'API)

**Test aggiunti** (`BuilderTestU.pas`):
- `TestLogExceptionWithoutFormatter`
- `TestLogExceptionWithStackTraceFormatter`
- `TestLogExceptionWithMessageAndTag`

---

### 3. Tag Opzionale con Default Configurabile

```delphi
// Default tag = 'main'
Log.Info('Messaggio');                    // tag = 'main'
Log.Info('Messaggio', 'CUSTOM');          // tag = 'CUSTOM'

// Configurato nel Builder
Log := LoggerProBuilder
  .WithDefaultTag('MYAPP')
  .WriteToConsole.Done
  .Build;

// Sub-logger con default tag
OrderLog := Log.WithDefaultTag('ORDERS');
OrderLog.Info('Nuovo ordine');            // tag = 'ORDERS'
```

---

### 4. Minimum Log Level Globale

Gate prima dell'accodamento - evita overhead per messaggi filtrati.

```delphi
Log := LoggerProBuilder
  .WithMinimumLevel(TLogType.Warning)  // Debug e Info filtrati
  .WriteToConsole.Done
  .Build;

Log.Debug('Ignorato');   // Non accodato - zero overhead
Log.Info('Ignorato');    // Non accodato - zero overhead
Log.Warn('Loggato');     // OK
```

**Vantaggi:**
- Nessun oggetto `TLogItem` creato per messaggi filtrati
- Gate globale indipendente dai log level degli appender

**Test aggiunto:**
- `TestWithMinimumLevel`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [danieleteti/loggerpro](https://github.com/danieleteti/loggerpro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
