---
trigger: always_on
description: Estas reglas aplican a todo el repositorio. Una instrucción más cercana a un
---

# Instrucciones de desarrollo de Kivgraph

Estas reglas aplican a todo el repositorio. Una instrucción más cercana a un
archivo puede añadir restricciones, pero no puede relajar los contratos de
integridad, compatibilidad o verificación descritos aquí.

## Mapa de instrucciones

Este archivo se lee siempre. Los demás cubren un directorio y se leen al
trabajar en él; ninguno repite lo que ya está aquí ni lo contradice.

|directorio|archivo|qué cubre|
|---|---|---|
|`internal/`|`internal/AGENTS.md`|carga de Go, la pasada, caché de hechos, grafo canónico, generaciones, configuración, procesos|
|`internal/mcp/`|`internal/mcp/AGENTS.md`|superficie de tools, `index_project`, la skill, coste en tokens|
|`internal/rustloader/`|`internal/rustloader/AGENTS.md`|`rust-analyzer scip`, identidad SCIP, sysroot, descubrimiento Cargo|
|`cmd/kivgraph/`|`cmd/kivgraph/AGENTS.md`|ayuda, registro, `index [--full]`, `clean`, `stop`, `ui`|
|`ts-worker/`|`ts-worker/AGENTS.md`|worker TypeScript e identidad cross-repository|
|`web/`|`web/AGENTS.md`|el visor: layout, dibujo, coste por fotograma|
|`landing/`|`landing/AGENTS.md`|landing y documentación de usuario: capas, paleta, SEO, iconos|
|`benchmarks/`|`benchmarks/AGENTS.md`|informes, corpus y auditorías|

Un invariante que se infringe desde otro directorio vive aquí, no en el archivo
del directorio que nombra: que `landing/` no entre en ningún bundle se rompe
editando `scripts/build-bundle.sh`.

## Identidad del proyecto

- Proyecto: `Kivgraph`.
- Módulo Go: `github.com/Luqueee/kivgraph`.
- Ejecutable principal: `cmd/kivgraph`.
- Worker TypeScript: `ts-worker/`, paquete privado `@kivgraph/ts-worker`.
- LadybugDB es el almacenamiento canónico; el HotSnapshot es una proyección
  derivada y no una fuente alternativa de hechos.
- Los identificadores históricos `LUQUE-####` del backlog no se renombran.

## Qué pregunta contesta cada tool de Kivgraph

Este bloque es el canal de enrutado portable: `CLAUDE.md` es un enlace a este
fichero, así que Oh My Pi y Claude Code lo cargan los dos sin que nadie lo pida.
El campo `instructions` del servidor dice lo mismo, y Zed no lo lee.

| la pregunta | la tool |
| --- | --- |
| quién llama a esto, qué referencia a esto | `find_references` |
| quién implementa un tipo o método | `find_implementations` |
| qué se rompe si lo cambio | `get_blast_radius` |
| qué alcanza esto hacia fuera | `trace_dependencies` |
| quién lo usa desde otro repositorio | `find_cross_repo_consumers` |
| dónde está declarado | `find_symbol` |
| no sé cómo se llama, qué archivos abro | `find_by_intent`, con `keywords` |
| qué hay declarado en este paquete | `get_file_outline` |
| dame el código de estos símbolos -- hasta `20` por llamada | `get_source` |
| ¿está el grafo al día? | `graph_status` |
| empieza un índice sin sostener una llamada larga | `start_index_project` |
| cómo va ese índice | `get_index_status` |

Las aristas las resuelven `go/types`, el checker de TypeScript y
`rust-analyzer`, no la coincidencia de nombres: una lista de referencias vacía
significa que **nadie lo llama**, no que no se encontró. `grep` no puede decir
eso, y tampoco distingue dos métodos homónimos.

Toda fila trae repositorio, ruta, nombre cualificado y rango de líneas, y toda
tool acepta esa tripleta en vez de una clave estable: la llamada siguiente se
construye con la respuesta que ya se tiene.

Una pregunta de referencias no necesita resolver el símbolo antes: `name` a
secas basta, y cuando varias declaraciones comparten el nombre la respuesta se
niega a elegir y **nombra los candidatos** con esa misma tripleta, así que
acotar es copiar uno. Sobre `workspace` la negativa cuesta `129` tokens donde el
`find_symbol` previo costaba `750`; medido en
`benchmarks/graft-comparison/report.md`.

Y se pide a la granularidad que se pregunta: `view: "files"` responde qué
archivos sin la línea de cada referencia. Las cuatro preguntas de referencias de
`workspace` cuestan `2.480` tokens con línea y `912` sin ella, con la misma precisión
y la misma exhaustividad -- y una página en vez de dos donde 66 referencias caben
en 9 archivos. Quien necesite la línea pide las filas compactas, que son el
valor por defecto.

**Dónde pierde, y conviene no gastar la llamada:** un nombre raro en un solo
repositorio pequeño lo resuelve `grep` más barato -una llamada, sin esquema,
sin resolver un símbolo primero-, y el índice de un fichero pequeño cuesta más
que leerlo. Gana en nombres comunes, en impacto transitivo, en consumidores de
otro repositorio y en demostrar una ausencia.

Medido con `benchmarks/mcp-token-cost` después del ADR 0046, sobre las seis
preguntas de referencias de este mismo repositorio (generación `000001`,
commit `f8a952d6`): responder cuesta entre `3,29x` (`MergeAll`, nombre raro) y
`11,95x` (`NewServer`, nombre común) menos que `grep` más la lectura; la sesión
completa -incluidos los cuerpos que el agente abre después, que pagan igual en
los dos lados- entre `1,26x` y `8,05x`, con un suelo de `2,41x` fijado por el
coste de esos cuerpos y no por el de la respuesta.

El caso genuinamente trivial **ya es una fila medida**, y confirma la
desventaja: `benchmarks/graph-tools-comparison/trivial.md`, sobre

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Luqueee/kivgraph](https://github.com/Luqueee/kivgraph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
