---
trigger: always_on
description: Usa siempre `/caveman full` (coincide con el hook global de SessionStart). Habla en español;
---

# Notas para agentes IA

Usa siempre `/caveman full` (coincide con el hook global de SessionStart). Habla en español;
los `.md` de documentación pueden ir en español.

## Que es esto

**FFXIVSpanish Patcher** es una app de escritorio para generar un mod `.pmp` de Penumbra con la
traduccion al castellano de Final Fantasy XIV.

Landing page publica del proyecto: <https://ffxivspanish.carrd.co/>. Se usa como `Website` en el
`meta.json` del `.pmp` generado.

La app:

1. Detecta o pide la ruta de una instalacion local de FFXIV.
2. Lee la version instalada del juego desde `ffxivgame.ver`.
3. Extrae solo las paginas `.exd` necesarias via Lumina; no dumpea el juego entero.
4. Aplica traducciones embebidas en el ejecutable.
5. Filtra/omite por defecto filas SeString inseguras y sigue empaquetando el resto.
6. Genera un `.pmp` instalable en Penumbra.

No modifica DATs ni otros archivos originales del juego. No versionar ni redistribuir bytes de
Square Enix (`.exd`, `.exh`, `.pmp`, dumps, snapshots reales).

La GUI es **.NET 10 + Avalonia UI (MVVM)**. La app es un shell fino; la logica de extraccion,
parcheo binario EXD, SeString y empaquetado vive en `src/FFXIVSpanishPatcher.Pipeline` y en codigo
vendorizado propio bajo `vendor/`.

## Estructura actual

- `src/FFXIVSpanishPatcher.App` - GUI Avalonia, MVVM, tema oscuro, iconos, update-check de GitHub
  Releases, version recomendada embebida y recursos `translations.dat`.
- `src/FFXIVSpanishPatcher.Pipeline` - orquestacion `load -> SeString gate -> extract -> patch ->
  package -> verify`, detector de ruta/version del juego, categorias, integridad y eventos
  `IProgress<PipelineEvent>`.
- `vendor/XivSpanish.Core` - modelos de traduccion, JSONL, manifest, normalizacion, claves de origen.
- `vendor/XivSpanish.GameData` - Lumina, EXH/EXD, `ExdPatcher`, lectura de filas, parser/tokenizer
  SeString, `GameLocator`.
- `vendor/XivSpanish.Packaging` - primitivas de empaquetado: broadcast, alias de campos,
  contamination guard y gate SeString de manifest.
- `tools/XivSpanish.BlobBuilder` - tool C# `sync`/`build` para regenerar traducciones embebidas y
  `data/recommended-game-version.txt`.
- `data/translations.dat` - blob Zstandard-JSONL versionado con filas empaquetables.
- `data/recommended-game-version.txt` - version de FFXIV para la que se construyo el blob/release.
- `data/translation-milestones.md` - historial Markdown embebido que muestra la GUI.
- `data/translations/jsonl/` - corpus crudo local, git-ignored; no se versiona.
- `tests/FFXIVSpanishPatcher.Tests` - tests de pipeline, EXD sintetico, categorias, detector, SeString
  y packaging.
- `tests/FFXIVSpanishPatcher.App.Tests` - tests headless de GUI/blob/viewmodel.
- `.github/workflows/ci.yml` - build + test en `main` y PR.
- `.github/workflows/release.yml` - tags `vX.Y.Z` publican zips self-contained para `win-x64`,
  `linux-x64` y `osx-arm64`.
- `build/macos/Info.plist` - metadatos del bundle `.app`.
- `docs/DESIGN.md` - diseno historico/arquitectura.
- `README.md`, `CONTRIBUTING.md`, `AI_USAGE.md`, `NOTICE.md` - docs publicas/legal/uso.

## `vendor/`: codigo propio

`vendor/` se sembro copiando partes de upstream **FFXIV-Spanish**, pero ahora es codigo mantenido en
este repo. Se puede editar. No hay resync automatico seguro.

No recrear `sync-vendor.ps1` ni sobrescribir `vendor/` desde upstream: destruiria divergencias ya
intencionales (modelo recortado, `gold`, SeString gate, packaging, codigo muerto eliminado, etc.).
Si hace falta una mejora upstream, portarla a mano y probarla aqui.

La procedencia historica vive en `vendor/VENDORED.md`.

## Decisiones cerradas

1. GUI = **.NET 10 + Avalonia UI**, no WPF.
2. Distribucion = ejecutables self-contained single-file por RID; no instalador obligatorio.
3. Traducciones = blob Zstandard embebido en el ejecutable single-file en todas las plataformas.
   `ExternalTranslations` queda solo como opcion diagnostica/local. Actualizar traducciones implica
   regenerar blob y publicar nueva release.
4. Datos del juego = extraccion lean desde la instalacion del usuario. No redistribuir archivos de FFXIV.
5. Corpus empaquetable = `status in {approved, gold}` + `target` no vacio + `sourceKey` util.
6. SeString = validar compatibilidad. Por defecto, filas incompatibles se omiten con warning; solo
   empaquetar con `ForceSeString` cuando se quiera diagnosticar conscientemente.
7. Categorias avanzadas = hibridas: catalogo/labels/tooltips curados en app, contadores y enablement
   derivados del manifest embebido.
8. Tests de integracion = EXD sintetico generado en codigo. Nunca depender del juego instalado en CI.
9. Legal/docs = mantener claro que el proyecto no esta afiliado a Square Enix y que las contribuciones
   de traduccion tienen terminos propios (`CONTRIBUTING.md`).

## Comandos

```bash
dotnet restore --locked-mode
```
```bash
dotnet build -c Release --no-restore
```
```bash
dotnet test -c Release --no-build
```

# Regenerar corpus crudo desde upstream y reconstruir blob + version recomendada.
```bash
dotnet run --project tools/XivSpanish.BlobBuilder -- sync --build
```

# Solo compactar data/translations/jsonl/ -> data/translations.dat.
```bash
dotnet run --project tools/XivSpanish.BlobBuilder -- build
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JadeArkadian/FFXIV-Spanish-Patcher](https://github.com/JadeArkadian/FFXIV-Spanish-Patcher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
