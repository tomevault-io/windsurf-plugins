---
trigger: always_on
description: Guia de trabajo para seguir construyendo el nodo comunitario `n8n-nodes-redmine`.
---

# AGENTS.md

Guia de trabajo para seguir construyendo el nodo comunitario `n8n-nodes-redmine`.

## Objetivo del paquete

Este repositorio implementa un nodo comunitario de n8n para consumir la API REST de Redmine. La superficie actual expone:

- Credenciales `Redmine API`, con `url` y `apiKey`.
- Recurso `Issue`, con operaciones `get`, `getAll`, `create`, `update`, `delete` y `addWatcher`.
- Recurso `Project`, con operaciones `get`, `getAll`, `create`, `update` y `delete`.
- Recurso `User`, con operaciones `get`, `getCurrent`, `getAll`, `create`, `update` y `delete`.
- Opcion global `Impersonate User`, enviada como header `X-Redmine-Switch-User`.

La referencia funcional principal es el `README.md`. Para detalles de endpoints y parametros, contrastar siempre contra la documentacion oficial de Redmine:

- https://www.redmine.org/projects/redmine/wiki/Rest_api

Version minima soportada de Redmine: `5.0`.

## Stack y comandos

- Runtime minimo: Node.js `>=18.10`.
- Package manager esperado: `pnpm@9.1.4`.
- Lenguaje: TypeScript, CommonJS, salida en `dist/`.
- Tests unitarios: Vitest.
- API n8n: `n8nNodesApiVersion: 1`.
- Flujo de release esperado: publicar en npm usando `pnpm prepublishOnly` antes de publicar.

Comandos habituales:

```bash
pnpm build
pnpm lint
pnpm test
pnpm format
pnpm dev
pnpm prepublishOnly
```

Antes de cerrar cambios de codigo, ejecutar como minimo:

```bash
pnpm build
pnpm lint
pnpm test
```

Si se tocaron descripciones, propiedades o estructura del nodo, tambien revisar manualmente que el nodo cargue en n8n y que los `displayOptions` muestren solo los campos esperados.

Los tests unitarios viven en `tests/` y usan Vitest. Priorizan validar el armado de requests, paginacion, filtros, payloads, headers, errores y `pairedItem` sin depender de una instancia Redmine real.

## Mapa del repositorio

- `credentials/RedmineApi.credentials.ts`: definicion de credenciales `redmineApi`.
- `nodes/Redmine/Redmine.node.ts`: descripcion principal del nodo, recursos, credenciales, opciones globales y dispatch de ejecucion.
- `nodes/Redmine/IssueOperations.ts`: agregador de propiedades para Issues.
- `nodes/Redmine/issue/operations.ts`: lista de operaciones de Issue.
- `nodes/Redmine/issue/fields.ts`: campos comunes, create y update de Issue.
- `nodes/Redmine/issue/additionalFields.ts`: campos adicionales de Issue.
- `nodes/Redmine/Issue.getAll.Operations.ts`: filtros y parametros especificos de `issue:getAll`.
- `nodes/Redmine/IssueExecute.ts`: ejecucion HTTP de operaciones de Issue.
- `nodes/Redmine/ProjectOperations.ts`: operaciones y campos de Project.
- `nodes/Redmine/ProjectExecute.ts`: ejecucion HTTP de operaciones de Project.
- `nodes/Redmine/UserOperations.ts`: operaciones y campos de User.
- `nodes/Redmine/UserExecute.ts`: ejecucion HTTP de operaciones de User.
- `nodes/Redmine/redmine.svg`: icono empaquetado por `gulp build:icons`.
- `tests/helpers.ts`: contexto fake de `IExecuteFunctions` para unit tests.
- `tests/IssueExecute.test.ts`: unit tests de operaciones de Issue.
- `tests/ProjectExecute.test.ts`: unit tests de operaciones de Project.
- `tests/UserExecute.test.ts`: unit tests de operaciones de User.
- `vitest.config.mts`: configuracion de Vitest, incluyendo alias para `n8n-workflow`.
- `package.json`: metadata n8n, scripts, keywords y archivos publicables.

## Patrones de implementacion

Mantener separada la declaracion UI de la ejecucion:

- Los campos del editor de n8n viven en archivos `*Operations.ts` o en subcarpetas especificas como `nodes/Redmine/issue/`.
- La logica que arma `endpoint`, `method`, `body` y `qs` vive en `*Execute.ts`.
- `Redmine.node.ts` solo debe orquestar recursos, credenciales, opciones globales y el dispatch hacia `execute*Operation`.

Para requests a Redmine:

- Usar `this.helpers.request(options)`.
- Enviar API key como header `X-Redmine-API-Key`.
- Enviar JSON con `Content-Type: application/json`.
- Construir URLs con `${baseUrl}/` + `endpoint.replace(/^\//, '')`.
- Si `body` queda vacio, eliminar `options.body`.
- Convertir errores HTTP en `NodeOperationError` con `{ itemIndex: i }`.
- Devolver siempre `{ json: responseData, pairedItem: { item: i } }`.
- Cuando `returnAll` sea `true`, traer todas las paginas que matcheen con los filtros. Manejar internamente el paginado de Redmine hasta agotar resultados.

Para impersonacion:

- Leer `options.impersonateUser`.
- Si existe, enviar `X-Redmine-Switch-User`.
- Documentar que requiere API key de admin.

Para colecciones de custom fields:

- En filtros de Issues, mapear a query params `cf_<id>`.
- En create/update, mapear a `custom_fields: [{ id, value }]`.

Para operaciones destructivas:

- Mantener `delete` como `DELETE /resource/:id.json`.
- No agregar confirmaciones artificiales dentro del nodo; n8n ya maneja la UX de ejecucion.

## Checklist para agregar una operacion

1. Confirmar endpoint, metodo, payload y respuesta en la API REST de Redmine.
2. Agregar la opcion en el `operations` del recurso correspondiente.
3. Agregar campos requeridos y opcionales con `displayOptions` precisos.
4. Implementar la rama en el archivo `*Execute.ts`.
5. Mantener nombres de parametros consistentes con Redmine cuando sean campos de API (`project_id`, `status_id`, etc.).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jonathanferreyra/n8n-nodes-redmine](https://github.com/jonathanferreyra/n8n-nodes-redmine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
