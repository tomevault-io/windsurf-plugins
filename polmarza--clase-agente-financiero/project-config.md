---
trigger: always_on
description: Archivo de referencia para cualquier agente de codificación que trabaje en este proyecto.
---

# CLAUDE.md

Archivo de referencia para cualquier agente de codificación que trabaje en este proyecto.
Lee este archivo completo antes de hacer cualquier cambio.

## Estado del proyecto y arranque

La documentación de `docs/` está **completa**. Antes de tocar nada:

1. Lee todos los archivos de `docs/` — incluida la subcarpeta `docs/criterio/`,
   que contiene el criterio financiero del sistema.
2. Lee `docs/roadmap.md` y localiza en qué fase está el proyecto.
3. Lee «Trampas conocidas del stack» en `docs/architecture.md`. Están ahí
   porque ya nos costaron tiempo: te ahorran repetirlo.

No empieces a escribir código sin haber hecho lo anterior. Si algo de la
documentación contradice lo que te pide el usuario, dilo antes de actuar.

---

## Descripción del proyecto

Asesor financiero conversacional. Una persona mantiene una conversación de
cinco minutos en lenguaje natural y recibe un plan financiero personalizado
escrito en castellano llano. La asesora ve todos sus clientes en un panel.

Dos piezas que no se mezclan: un modelo de lenguaje que entrevista y redacta, y
un motor de cálculo determinista que hace todas las cuentas.

**Nombre:** clase-agente-financiero
**Descripción:** Entrevista financiera conversacional con diagnóstico calculado y plan en lenguaje llano.
**Estado actual:** En desarrollo — ver `docs/roadmap.md`

---

## Documentación de referencia

| Archivo | Qué contiene |
|---|---|
| `docs/prd.md` | Qué construimos y para quién |
| `docs/business.md` | Modelo de valor y **restricciones regulatorias** |
| `docs/architecture.md` | Decisiones técnicas y trampas del stack |
| `docs/data-model.md` | Tablas y contrato de la ficha |
| `docs/design-system.md` | Tono, color y componentes |
| `docs/roadmap.md` | Fases y criterios de aceptación |
| `docs/user-flows.md` | Recorridos de cliente y asesora |
| `docs/testing.md` | Estrategia de pruebas |
| `docs/criterio/` | **El criterio financiero.** Reglas R1–R10, plantilla de entrevista, instrucciones del motor |

**Orden de autoridad:** si `docs/criterio/reglas-recomendacion.md` contradice a
cualquier otro documento, manda el criterio. Es la única fuente de verdad
financiera. `docs/criterio/politica-inversion.md` está **derogado**: no lo leas.

---

## Stack tecnológico

- Framework: Next.js 16 (App Router) + TypeScript
- Base de datos: Supabase (PostgreSQL + Auth), región europea
- Estilos: Tailwind CSS
- Modelo de lenguaje: API de Anthropic (`claude-sonnet-5`), solo desde servidor
- Gráficos: Recharts
- Tests: Vitest
- Despliegue: Vercel

---

## Estructura de carpetas

```
src/
├── app/            → rutas (App Router) y rutas de servidor en app/api/
├── components/     → componentes reutilizables
├── lib/
│   ├── motor/      → MOTOR DE CÁLCULO VERIFICADO. No tocar.
│   ├── supabase/   → clientes de base de datos
│   └── claude/     → prompts y definición de herramientas
└── types/          → tipos compartidos

docs/               → documentación del proyecto
└── criterio/       → el criterio financiero heredado
motor-python/       → motor original + baseline (oráculo de los tests)
supabase/           → migraciones
material-clase/     → material didáctico
changelog/          → registro de cambios
mejoras/            → ideas futuras no implementadas
```

---

## Convenciones de código

- Gestor de paquetes: pnpm v11. No usar npm ni yarn.
- TypeScript estricto. No usar `any`.
- Idioma de comentarios, nombres de dominio y textos: **español**.
- Los nombres del dominio financiero se escriben como en el criterio:
  `flujoLibre`, `aportacionPropuesta`, `colchonMeses`. No se traducen al inglés,
  para que la trazabilidad con `docs/criterio/` sea directa.
- Nombrado de componentes: PascalCase. Archivos: kebab-case.
- Los comentarios explican **por qué**, no qué. El qué ya lo dice el código.

---

## Qué NO hacer

### Del dominio — rompen el producto

- **El modelo de lenguaje no calcula nunca.** Entrevista y redacta. Todo número
  sale del motor. Si te ves escribiendo una cuenta dentro de un prompt, párate.
- **No inventes ni completes datos del cliente.** Lo que no dé, se etiqueta
  `pendiente`. Nunca un número plausible.
- **No nombres productos concretos.** «Un fondo indexado mundial» como
  categoría, sí. Una gestora, un fondo o un ticker, jamás.
- **No prometas rentabilidades.** Siempre horquillas y probabilidades con sus
  supuestos declarados.
- **No subas el riesgo para que una meta cuadre.** Única excepción, la de R4.
- **No omitas el descargo** de orientación educativa en ningún plan emitido.

### Del código

- **No toques `src/lib/motor/`.** Es un port verificado con 95 tests. Si crees
  que hay que cambiarlo, pregunta antes. Si un test del motor falla, el fallo
  está en lo que lo rodea, no en él.
- **No dupliques valores de criterio.** Viven solo en
  `src/lib/motor/supuestos.ts`. Si cambia una regla, se cambia ahí y se
  regenera el baseline con `pnpm baseline`.
- **No desactives un test para que pase el build.**
- **No crees políticas RLS para el rol `anon`.** El cliente nunca habla con la
  base de datos.
- **No pongas `NEXT_PUBLIC_` a `SUPABASE_SERVICE_ROLE_KEY`.** Ese prefijo
  empaqueta la variable en el JavaScript del navegador.
- **No sobrescribas una ficha cerrada.** Se versiona.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [polmarza/Clase-Agente-Financiero](https://github.com/polmarza/Clase-Agente-Financiero) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
