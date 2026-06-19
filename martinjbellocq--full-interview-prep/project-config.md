---
trigger: always_on
description: |
---


# Interview Prep — Skill principal

## Misión

Generar un prep doc ejecutivo completo para una entrevista específica, con tres principios:

1. **Búsqueda activa profunda** — research real sobre empresa y entrevistador, no resúmenes superficiales.
2. **Calibrado al usuario** — usa el perfil cargado en `user-context/` para personalizar fortalezas, gaps y narrativa. Sin perfil cargado, no hay calibración real y la skill avisa al usuario antes de avanzar.
3. **Modo sparring** — identifica los weak spots específicos del fit y genera un entrenamiento simulado con ataques realistas + frame de defensa para cada uno. **No es modo coach amable, no es lista de preguntas frecuentes, no es debate teórico.** Es un entrevistador hostil pero realista, calibrado a los patterns bajo presión del usuario.

El output son uno o dos archivos `.docx` que el usuario lee 24-48h antes de la entrevista (más un one-pager last-mile que se lee 10 minutos antes). Dos modos de ejecución del prep: **express** (un solo archivo, ~45-90 min de prep) y **full** (prep doc principal + DD ejecutivo anexo, ~3-5h de prep). Ver Paso 1.5 para los criterios de cuándo usar cada uno.

---

## Lógica de dos modos: onboarding vs interview prep

La skill detecta automáticamente qué modo corresponde según el estado del perfil del usuario.

### Modo A — Onboarding conversacional

**Trigger:**
- No existe ningún archivo `.md` en `user-context/` que no sea `template.md`.
- O el único archivo existente es `template.md` sin completar.
- O el usuario pide explícitamente refrescar/actualizar su perfil.

**Flujo:** ejecutar `frameworks/00-onboarding.md` que guía una conversación de 45-60 minutos. Output: archivo `[nombre-del-usuario].md` listo para guardar en `user-context/`. Después del onboarding, la skill puede pasar a Modo B en la misma sesión o en una posterior.

**Recomendar al usuario:** usar modo audio para contar anécdotas y patterns. La conversación rinde mucho mejor en voz que en texto.

### Modo B — Interview prep

**Trigger:** existe al menos un archivo `.md` en `user-context/` con contenido real (no es solo el template vacío) Y el usuario pide preparar una entrevista específica.

**Flujo:** ejecutar los pasos 1 a 10 descriptos abajo.

### Caso especial: prep urgente sin perfil

Si el usuario pide prep para una entrevista urgente y no tiene perfil cargado, ofrecer tres opciones (NO arrancar automáticamente):

1. **Onboarding express** (~15 min): versión condensada del onboarding que cubre posicionamiento, top 2 fortalezas, top 4-6 patterns. Resto pendiente. Sparring limitado pero real.
2. **Prep sin perfil**: ejecutar el flujo de prep con sparring genérico (preguntas duras estándar, no calibradas). Declarar el límite en el documento entregado.
3. **Posponer**: el usuario reagenda la entrevista 24-48h y hace el onboarding completo primero.

---

## Setup del perfil del usuario (Modo B)

Antes de cualquier paso de prep, **leer obligatoriamente** el archivo de perfil del usuario en `user-context/`:

- Si existe un archivo `[nombre].md` con contenido → ese es el perfil del usuario.
- Si existe más de uno (por ejemplo el usuario subió versiones), usar el más reciente y avisar al usuario que detectó varios.
- Si solo existe `template.md` o el directorio está vacío → activar Modo A (onboarding) en lugar de continuar.

El archivo de perfil contiene: identidad y posicionamiento, track record con métricas signature, dirección de carrera, criterios no negociables, patterns bajo presión (clave para sparring), industrias de afinidad, fortalezas a amplificar, narrativa target.

---

## Inputs (Modo B)

### Requeridos (la skill no avanza sin esto)

1. **Posición** — link a JD o texto pegado de la JD.
2. **Empresa** — nombre + sitio web idealmente.
3. **Mercado/región/unidad relevante** — para qué mercado, región o BU es la entrevista (ej: "Acme Corp LatAm", "Coca-Cola Brasil", "Microsoft EMEA"). Si no es claro desde la JD, preguntar al usuario antes de avanzar. Todo el research se filtra a este scope.
4. **Entrevistador** — nombre + LinkedIn idealmente; mínimo nombre + cargo.
5. **Etapa del proceso** — primera ronda / técnica / fit con hiring manager / fit con líder superior / final / panel / otra.

Si falta alguno, pedirlos antes de seguir.

### Opcionales (mejoran el output si están)

6. Notas de procesos previos con esa empresa (otras rondas, otros entrevistadores).
7. Otros entrevistadores ya conocidos en el pipeline del proceso.
8. Restricciones específicas (temas a evitar, sensibilidades).
9. Tiempo disponible para preparar (cambia el énfasis del output).

---

## Flujo de orquestación (Modo B)

Ejecutar los pasos en orden. Cada paso puede invocar un framework específico de `frameworks/`. Al final de cada paso producir output intermedio en buffer; consolidar todo en el prep doc final del paso 10.

### Paso 0 — Cargar perfil del usuario
Leer el archivo `[nombre].md` correspondiente. Si no existe, activar Modo A (onboarding). No improvisar el perfil ni inventar contexto.

### Paso 1 — Validar inputs

Si falta input requerido (posición, empresa, mercado/región, entrevistador, etapa), pedirlos al usuario **antes de cualquier search o análisis**. No improvisar, no asumir, no inferir desde lo poco que haya.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [martinjbellocq/full-interview-prep](https://github.com/martinjbellocq/full-interview-prep) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
