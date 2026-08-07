---
trigger: always_on
description: App PWA de cálculo de nómina para trabajadores por turnos en Colombia. Vanilla JS sin build obligatorio, React via CDN UMD, Supabase como backend. La documentación técnica completa para onboarding está en `ARCHITECTURE.md`.
---

# Mi Turno — guía para agentes

App PWA de cálculo de nómina para trabajadores por turnos en Colombia. Vanilla JS sin build obligatorio, React via CDN UMD, Supabase como backend. La documentación técnica completa para onboarding está en `ARCHITECTURE.md`.

## Stack

- **Frontend**: vanilla JavaScript ES5-flavored, React 18 (UMD globals: `React`, `ReactDOM`), `h = React.createElement` (no JSX, no transpilación).
- **Persistencia local**: `localStorage` via `leer()`/`grabar()` (en `js/utils/storage.js`).
- **Backend**: Supabase (auth + Postgres + Realtime). Project ID: `yrpqvmqmchsxpotytxiy`.
- **PWA**: Service Worker (`sw.js`) con precache estricto. Cada archivo nuevo necesita registro manual ahí.
- **Deploy**: Vercel (auto-deploy desde `master`).
- **i18n**: español, mezcla tú/vos (vos en frases emocionales, tú en setup formal). UI en español argentino/colombiano.

## Convenciones de código (NO romper)

- Usar `var`, NUNCA `let`/`const` (excepto en `sw.js`/`init.js` que sí usan `const`).
- Usar `function (x) { return ... }`, NUNCA arrow functions (`x => ...`) — rompe la consistencia con el codebase y v37 ya tuvo que revertir una.
- Usar `h(tag, props, ...children)`, NUNCA JSX.
- `async`/`await` y `Promise` SÍ están permitidos (usados en `services/sync-queue.js`, `services/export-email.js`, `utils/otp.js`, `utils/password-hash.js`). Mantener `.then(function (res) {})` cuando la consistencia con el archivo lo requiera, pero `async function foo() { var x = await bar(); }` es válido.
- Sin emojis en código a menos que el usuario los pida explícitamente para UX.
- Comentarios solo cuando el **porqué** no es obvio; nunca explicar **qué** hace el código.

## Carga de scripts

- Todo es global (`window.*`). Orden de `<script>` en `app.html` importa.
- Al agregar un archivo nuevo: hay que ponerlo en **`app.html`** Y en **el array `appResources` de `sw.js`** (precache). Si falta en sw.js, falla offline. Hay un linter rudimentario en `scripts/check.sh`.

## Versionado (las 3 fuentes de verdad)

Tres archivos tienen que decir lo mismo en cada release:

| Archivo | Línea |
|---|---|
| `js/config/globals.js` | `var MT_APP_VERSION = 'vNN';` |
| `sw.js` | `const SHELL_CACHE = 'mt-shell-vNN';` |
| `version.json` | `"v": "vNN"` |

**Usá siempre `scripts/bump.sh NN "Label"` para bumpear** — el sed sincroniza las 3. Hacerlo a mano causó el bug del bucle nuclear infinito en v28 (sw.js iba a v28, globals.js seguía en v27, el detector de reload fantasma loopeaba). `scripts/check.sh` aborta si están desincronizados.

## Datos: qué es local-only vs nube

| Dato | Local | Nube | Notas |
|---|---|---|---|
| Turnos cerrados | `mt_t_<uid>` *(via `dk(uid,"t")`)* | `public.turnos` | Realtime ✓ |
| Turno activo | `mt_a_<uid>` *(via `dk(uid,"a")`)* | `public.turno_activo` | Realtime ✓ (v39) |
| Salario base | `mt_s_<uid>` *(via `dk(uid,"s")`)* | `public.perfiles.salario_base` | Local manda si `sc=true` (v27) |
| Flag salario configurado | `mt_sc_<uid>` *(via `dk(uid,"sc")`)* | — | Local-only por device |
| PIN | `mt_pin_<uid>` | `public.pin_lookup` | PK=pin, UNIQUE(user_id). **Upsert SIEMPRE con `{onConflict:'user_id'}`** (v36) |
| Email | — | `auth.users.email` + `pin_lookup.user_email` + `perfiles.email` | Cambiar email requiere update en cascada (v36) |
| Password | — | `auth.users` | Solo online |
| Profile name (alias "pipe") | `mt_pname_<uid>` *(via `dk(uid,"pname")`)* | — | Local-only por device (v30) |
| Profile photo | `mt_photo_<uid>` *(via `dk(uid,"photo")`)* | — | Local-only, JPEG 240×240 base64 (v30) |
| Prefs (quincenaMode, etc.) | `mt_prefs_<uid>` *(via `dk(uid,"prefs")`)* | — | Local-only |
| Sesión actual | `mt_sess` | — | Limpia en logout (preserva PIN+offline cache) |
| Marcador "device conocido" | `mt_last_user` | — | Habilita FastPinScreen (v37) |
| Sesión offline cacheada | `mt_offline_<base64(email)>` | — | Para login sin red |
| Password offline | `mt_pass_<base64(email)>` | — | PBKDF2-SHA256 + salt random (v49). Schema `{v,s,h}`. Verificación legacy plaintext para migración suave |

## Flujo de sync (offline-first)

```
Usuario hace acción
  → setX local (UI instantánea)
  → queueAction(uid, type, payload)         [sync-queue.js]
  → _scheduleFlush(uid)                     [debounce 250ms]
  → processQueue(uid)                       [IN_FLIGHT guard]
  → supaXxx() helpers                       [supabase.js]
  → Realtime channel del propio user_id dispara
  → otros devices reciben → cargarDatos() → setX
```

**Contrato crítico**: `queueAction()` MUST trigger eventual `processQueue()`. Sin el `_scheduleFlush` la cola se estancaba (v40 fix — bug que persistió varias releases porque atacaba el síntoma equivocado).

**Realtime**: solo habilitado para `turno_activo` y `turnos` (publication `supabase_realtime`). Para agregar otra tabla:
```sql
ALTER PUBLICATION supabase_realtime ADD TABLE public.<tabla>;
ALTER TABLE public.<tabla> REPLICA IDENTITY FULL;
```

## Schemas de Supabase

```
auth.users           (uid PK)              ← fuente de verdad de email + password
public.perfiles      (id PK = uid)         email, salario_base, updated_at

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aedincen-rgb/mi-turno](https://github.com/aedincen-rgb/mi-turno) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
