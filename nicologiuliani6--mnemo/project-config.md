---
trigger: always_on
description: Riferimento completo — subset C di Mnemo, cosa manca rispetto al C standard e limiti (da c_lower.py)
---


# Mnemo — subset C vs C “normale”

Mnemo compila un **sottoinsieme** di C verso **Kairos** (`mnemo/c_lower.py`, `mnemo/layout_collect.py`, `mnemo/compile.py`). Non è un compilatore C conforme allo standard. **Backlog e priorità**: vedi [`TODO.md`](../../TODO.md) nella root del repo. Questa nota elenca **limiti e differenze** rispetto al C d’uso comune.

## Tipi fondamentali

- **`char` / `signed char` / `unsigned char`** come tipo scalare di variabile: gli scalari ammessi sono sostanzialmente `int`, `unsigned` / `unsigned int`, `bool` / `_Bool` (`_SCALAR_NAMES`).
- **`short`, `long`, `long long`** (e varianti unsigned) come tipi distinti per dichiarazioni normali.
- **`float`, `double`, `long double`** e tipi complessi.
- **`wchar_t`, `size_t`, `ptrdiff_t`** come tipi “veri” del C standard (non c’è ecosistema ABI completo).

## Letterali e stringhe

- Letterali interi: `_const_int` accetta solo tipi AST `int`, `long`, `unsigned int`, `long long`.
- **Stringhe `"..."`** e uso “da manuale C” di array di caratteri / puntatori a stringa read-only.
- **Costanti carattere** `'x'` come primarie intere ovunque (percorso ristretto).

## Dichiarazioni e puntatori

- Puntatori dichiarati nel subset: **`int *`**, **`unsigned *` / `unsigned int *`**, **`void *`** (un livello, dove previsto), e **`T *`** con **`T`** typedef di **`struct`** / **`union`** (un livello; stesso modello “handle” dei parametri). Più locali puntatore (es. più `malloc`) sono supportati tramite pool `__mn_pool_*`.
- **`char *`** con stringa letterale `"..."` e **`const char *`** con la stessa convenzione sono supportati per `printf`/`putchar` nel subset; non è il modello puntatori/stringhe del C completo.
- **`const`, `volatile`, `restrict`, `_Atomic`**: spesso **accettati sintatticamente** (qualificatori su `TypeDecl`); **nessuna** semantica C completa (nessun enforcement const-correctness).
- **`static` / `extern`**: Mnemo usa **una TU** verso Kairos; linkage e durata non coincidono con il C standard su più unità di traduzione.

## `main`, `argc`, `argv`

- **`main`**: solo nessun parametro (`void`), solo **`int argc`**, oppure **`int argc`** poi **`char **argv`**; altre firme → errore.
- **`argv`** è **stub** (non array di stringhe POSIX, non dereferenziabile come in C reale).
- Nessun terzo argomento (`envp`), `wmain`, ecc.

## Array

- **VLA**: dimensione deve essere **costante intera** a compile-time.
- Prodotto dimensioni ≤ **`ARR_MAX` (1024)** elementi totali.
- Elementi: solo **scalari Mnemo** o **puntatore** nel senso sopra (`int*`, `void*`, ecc.).

## Struct e union

- **Passaggio struct per valore** (flatten sui campi) è supportato dove previsto dal layout; **union** per valore come singola parola.
- **Inizializzatori `{ ... }`** alla dichiarazione: **struct** (lista in ordine di campi, senza designatori); **union** con **un** solo valore in `{ … }`.
- **`struct.campo`**: `=` e assegnamenti composti (`+=`, `*=`, `^=`, `<<=`, … come per gli scalari).
- **`p->campo`**: idem (`=` e composti).
- **Union**: assegnamenti composti solo per un sottoinsieme di operatori; altri → errore.
- **Bit-field** nei struct: non nel modello (campi appiattiti a interi).

## Enum

- Enum come **costanti intere**; `switch`/`case` con costante intera o enumeratore — non sistema tipi enum completo del C.

## Operatori unari

- Implementati in forme limitate: `-`, `*`, `&` (solo **`&id`** e **`&struct.campo`** con vincoli), **`~`** (come `(-1) ^ x`), `sizeof`.
- **`++` / `--`** (prefisso/postfisso) su **`ID`**, **`*p`**, **`a[i]`**, **`s.campo`**, **`p->campo`** (istruzione ed espressione), con ordine di side-effect allineato agli assegnamenti esistenti.
- Altri unari → `operatore unario non supportato`.

## Operatori binari e logici

- Espressioni: insieme chiuso di operatori (`+ - * / %`, `& | ^`, `<< >>`, virgola, ecc.); il resto → `operatore binario non supportato`.
- **`&&` / `||` / `==` / …** non sono valutati come nel C su ogni sotto-espressione: il controllo usa predicati dedicati (`if`, confronti atomici, `!` sul predicato).

## Assegnamenti

- Su **`ID`** scalare: `=`, `+=`, `-=`, `*=`, `/=`, `%=`, `^=`, `&=`, `|=`, `<<=`, `>>=`.
- **Non supportati** su scalare: altri composti → `assegnamento con … non supportato`.
- **`array[i]`** e **`*p`**: **`=`** e assegnamenti composti (stesso insieme di operatori degli scalari).
- Molti lvalue → `lvalue non-ID non supportato`.

## Cast

- Solo se `_cast_accepts_pointer_or_scalar`: **`void`**, puntatori **`void*` / `int*` / `T*`** (`T` struct o union typedef), scalari del subset; altro → `cast non supportato`.

## Indirizzo `&`

- Solo **`&variabile`** e **`&struct.campo`** (con disponibilità slot); non `&a[i]`, `&(expr)` generico, ecc.

## Puntatori e memoria

- Semantica **indici pool** / celle **`__mn_mem*`**, non aritmetica puntatori e confronti del C pieno.
- **`malloc` / `free`**: modello **pool** dimensionato dalla toolchain, non libc.

## Chiamate

- Callee: nome diretto **`f(...)`**, oppure **puntatore a funzione** risolto a **compile-time**: variabile dichiarata `int (*p)(…)` con init **`g`** o **`&g`** (`g` dichiarata nello stesso file), più **`p(...)`** e **`(*p)(...)`**. Nessun puntatore a funzione calcolato a runtime generico.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nicologiuliani6/mnemo](https://github.com/nicologiuliani6/mnemo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
