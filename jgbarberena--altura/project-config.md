---
trigger: always_on
description: > Contexto transversal del proyecto. Para trabajo en el panel de administración lee también `CLAUDE_ADMIN.md`. Para trabajo en el frontend público, `CLAUDE_WEB.md`.
---

# CLAUDE.md — experienciasanfermin.com

> Contexto transversal del proyecto. Para trabajo en el panel de administración lee también `CLAUDE_ADMIN.md`. Para trabajo en el frontend público, `CLAUDE_WEB.md`.

> **Historial de deudas cerradas:** `CLAUDE_ADMIN_BACKLOG.md`. Ese archivo NO se carga en sesiones normales. Para añadir una deuda resuelta al backlog sin cargarlo: `Add-Content -Path 'CLAUDE_ADMIN_BACKLOG.md' -Value '...' -Encoding UTF8`.

---

## 1. Personas

**Javier** (jgbarberena@gmail.com) — desarrollador no profesional con conocimientos sólidos de programación (clases, funciones, variables, contratos, dependencias). Ha programado en VBA, HTML, JS, C++, Matlab, Mathematica y Modelica. Entiende el código cuando lo lee y sabe expresar con precisión lo que quiere. Es quien trabaja con Claude Code.

**Paula Díaz** (paula@experienciasanfermin.com / +34 625 638 977) — usuaria principal del panel de administración. Gestiona clientes por WhatsApp y correo, principalmente desde el móvil. Firma los documentos comerciales (NIF: 72694758S). No es técnica.

**Ander Sagardia** (sistemas@ycgdigitalgroup.com, YCG Digital Group) — gestiona DNS y hosting del servidor de producción.

**Hilario** (goviwebs.com) — desarrollador de tienda.sanfermin.com (WooCommerce). Contacto para cambios en la API sfcom y configuración de productos en la tienda.

---

## 2. Cómo quiero que me ayudes

- Guíame paso a paso con avance real y verificable. No me des todo de golpe.
- Cuando algo falla: diagnóstico primero (qué error exacto, dónde), luego solución mínima y concreta, luego verificación.
- Sé directo y honesto. Si hay una decisión que reconsiderar, dímelo aunque yo haya propuesto algo diferente.
- Cuando tengo que hacer algo en una interfaz externa (Supabase, GitHub, etc.), dime exactamente a qué pantalla ir, qué menú abrir y dónde hacer clic.
- Si ves deuda técnica, dímelo aunque no lo haya preguntado, pero sin insistir si decido dejarlo para después.
- Sin formateo excesivo: prosa cuando explicas, código limpio cuando programas, sin bullets innecesarios.
- No asumir que entiendo una herramienta nueva sin explicar antes qué es y cómo funciona.
- No proponer soluciones que funcionen solo para un caso particular sin pensar en la arquitectura general.

**Al inicio de cada sesión:** si Javier no indica en qué va a trabajar, pregunta si es trabajo en el panel de admin o en la web pública, para cargar el contexto adicional correcto (CLAUDE_ADMIN.md o CLAUDE_WEB.md).

**Flujo de trabajo:**
- Algo nuevo: primero diseño y decisiones (preferiblemente en claude.ai para análisis extenso), luego implementación en orden lógico (dependencias primero), luego verificación con datos reales.
- Algo que falla: diagnóstico → solución mínima → verificación.

---

## 3. El proyecto

Web de reservas de balcones y experiencias para San Fermín en **experienciasanfermin.com** (también vivesanfermin.com), con panel de administración privado para uso propio.

**Volumen:** menos de 200 reservas, menos de 100 proveedores, 2-3 usuarios del panel.

**Repositorio:** https://github.com/jgbarberena/altura  
**GitHub Pages:** https://jgbarberena.github.io/altura/ (dominio propio: experienciasanfermin.com)  
**Deploy adicional:** FTP a servidor externo. Credenciales en `.vscode/sftp.json` — no commitear cambios.  
**Entorno local:** Live Server (VSCode).

---

## 4. Stack técnico

| Capa | Tecnología |
|---|---|
| Frontend público | HTML/CSS/JS puro, sin frameworks |
| Panel de admin | Páginas HTML bajo `/admin/`, JS en módulos ES6, acceso directo a Supabase desde el navegador |
| Base de datos | Supabase (PostgreSQL), @supabase/supabase-js@2 vía CDN |
| AI | Claude API vía Edge Function `claude-proxy` en Supabase. API key en Supabase Vault. Modelos disponibles: `claude-sonnet-4-6` (default), `claude-opus-4-7`, `claude-haiku-4-5-20251001`. |
| Email inbound | Resend (dominio verificado). Inbound webhook para `in.experienciasanfermin.com`. |
| Maps | Leaflet 1.9.4 |
| Excel export | SheetJS (carga dinámica vía `import()`, ~900KB, solo al primer uso) |
| PDF | `window.print()` o jsPDF para propuestas y facturas desde el panel |
| Hosting | GitHub Pages + FTP a servidor externo |

No hay servidor propio. No hay proceso de build, bundler ni transpilación. Lo que ves es lo que se sirve.

---

## 5. Deploy

El script `deploy.ps1` (raíz del proyecto) automatiza el ciclo completo: regenera índices + SEO + sitemap → git commit/push → FTP de los archivos modificados.

**Ejecución:**
```powershell
pwsh deploy.ps1 -Message "descripción breve"
```

Usar siempre `pwsh` (PowerShell 7), no `powershell` (PS5).

**Opciones:**
- `-SkipScripts` — omite regeneración de índices/SEO/sitemap (solo commit + FTP)
- `-SkipFtp` — solo commit/push, sin FTP
- `-SkipGit` — solo FTP, sin commit

**Claude ejecuta el deploy directamente** cuando Javier lo pida ("haz el deploy", "sube los cambios"). Flujo:
1. Revisar qué se ha modificado en la conversación.
2. Redactar mensaje de commit breve en español (máx. 60 caracteres).
3. Ejecutar vía Bash:
   ```powershell
   pwsh -ExecutionPolicy Bypass -File deploy.ps1 -Message "<mensaje>"
   ```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jgbarberena/altura](https://github.com/jgbarberena/altura) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
