---
trigger: always_on
description: Plataforma para gestionar los torneos de una liga de fútbol amateur de pueblo.
---

# Liga de Fútbol Amateur Fugaz (LDFAF)

Plataforma para gestionar los torneos de una liga de fútbol amateur de pueblo.
Cliente real, tráfico bajo.

## Estado del proyecto

**Fase actual: demo para presentar al cliente.** Debe verse bien y funcionar de verdad —
el objetivo es que el cliente valide que es lo que busca. Tras su aprobación se compra
dominio y se pasa a producción con base de datos real.

## Lo que pidió el cliente

Textualmente, por WhatsApp:

1. *"Quiero que el dirigente entre y se inscriban a sus jugadores, ponga el logo del club
   y nombre y apellidos, edad, número de cédula."*
2. *"Coloque el torneo: Copa Fugaz Fútbol 7, Copa Fugaz Fútbol 11, Copa Fugaz categoría
   sub 12... Que haya alguna categorías. Desde las sub 8 hasta las sub 40."*
3. *"Que el dirigente pueda inscribir a sus jugadores hasta una fecha límite que yo ponga
   como organizador."*

De ahí salen los **dos tipos de usuario** del sistema:

| | Organizador (el cliente) | Dirigente (cada club) |
|---|---|---|
| Crea torneos y categorías | ✅ | — |
| Fija la fecha límite de inscripción | ✅ | — |
| Ve los inscritos y sus nóminas | ✅ | solo la suya |
| Registra el club | — | ✅ |
| Carga la nómina de jugadores | — | ✅ |

La competición (clasificación, calendario, goleadores) es la fase siguiente, que arranca
cuando el torneo ya tiene equipos inscritos.

## Identidad visual

Tomada del escudo que envió el cliente (`assets/img/escudo.png`). Diseño **mobile-first**
y **solo tema oscuro** — es la identidad de la marca, no una preferencia del usuario.

**Los colores se extrajeron del PNG, no se inventaron.** Se midió el tono dominante de
cada uno de los cinco cuadros del escudo y se aclaró lo justo para que contraste sobre
fondo oscuro:

| Cuadro | Tono | Variable CSS | Valor |
|---|---|---|---|
| L | 133° verde | `--ldfaf-verde` | `#28ad46` |
| D | 214° azul | `--ldfaf-azul` | `#2861ad` |
| F | gris | `--ldfaf-grafito` | `#2c2c2e` |
| A | 267° morado | `--ldfaf-morado` | `#6528ad` |
| F | 2° rojo | `--ldfaf-rojo` | `#ad2c28` |

El **color principal de acción** es `--neon: #34c85c`, el verde del escudo aclarado
(contraste 9.0:1 sobre el fondo). Una versión anterior usaba un verde lima neón
(`#9ee641`) que no estaba en el logo: venía de una infografía de referencia. Cambiar el
color principal es tocar una sola variable.

### Los archivos del escudo

| Archivo | Tamaño | Dónde se usa |
|---|---|---|
| `assets/img/escudo.png` | 512×512, 175 KB | portada |
| `assets/img/escudo-mini.png` | 128×128, 15 KB | cabecera |

Salen del original de 1254×1254 y 1,2 MB, reducidos para no castigar el móvil.

⚠️ **El PNG no tiene transparencia**: su fondo es negro sólido. Por eso los contenedores
`.marca__escudo` y `.hero__escudo` van con `background: #000`, para que no se recorte una
caja negra sobre el fondo del sitio. Si el cliente envía una versión con fondo
transparente, se puede quitar ese negro.

## Flujo de trabajo

```
requisitos → implementación → validación → commit/push → GitHub → deploy
```

La validación va **antes** del commit. No se pushea nada sin comprobar que funciona.

## Stack

| Capa | Ahora (demo) | Producción (tras aprobación) |
|---|---|---|
| Código | GitHub — `SantyCloud/copa-fugaz` (**público**) | igual, valorar volverlo privado |
| Front | HTML/CSS/JS puro, sin build | igual |
| Datos | JSON en `data/` + `localStorage` | Supabase (Postgres) |
| Auth | `data/usuarios.json` + `sesion.js` (**solo demostración**) | Supabase Auth: organizador y dirigentes |
| Ficheros | — | Supabase Storage (logos de club) |
| Permisos | — | Supabase RLS |
| Deploy | Cloudflare Pages o GitHub Pages | + dominio propio |

El repo se hizo público a petición del cliente para poder usar GitHub Pages en plan
gratuito. Cloudflare Pages también sirve y acepta repos privados sin coste.

## ⚠️ Datos personales — regla crítica

El sistema guarda **nombre, apellidos, edad y número de cédula**, y hay categorías desde
**Sub-8**: son datos personales de menores de edad.

- **Todas las cédulas del repo son inventadas.** No corresponden a personas reales.
- **Nunca commitear datos reales de jugadores.** El repositorio es público: un commit con
  cédulas auténticas quedaría expuesto y cacheado para siempre aunque se borre después.
- En producción, esos datos van en Supabase con RLS, nunca en el repositorio.

## Modelo de negocio

Es un **SaaS de gestión de torneos**, no la web de una sola liga:

- **Organizar un torneo es de pago.** Crear torneos, abrir categorías y fijar los plazos
  requiere una membresía activa.
- **Inscribir un club es gratis.** El organizador da el acceso a cada dirigente.
- **La liga es pública.** Clasificación, calendario, clubes y goleadores se ven sin cuenta.

⚠️ **La venta no es la prioridad ahora.** La portada es la del campeonato del cliente:
enseña sus torneos y sus plazos, y lleva a inscribir un club. Contratar una membresía para
organizar torneos propios existe en `#/planes`, pero solo se llega desde el pie. El panel
del organizador sigue exigiendo membresía activa.

⚠️ **Los precios de `data/planes.json` son de ejemplo.** Hay que confirmarlos con el
cliente antes de presentarlo. Están en un JSON aparte justo para poder cambiarlos sin
tocar código.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SantyCloud/copa-fugaz](https://github.com/SantyCloud/copa-fugaz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
