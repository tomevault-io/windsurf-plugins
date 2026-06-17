---
trigger: always_on
description: CLI para interactuar con la sede electrónica de la AEAT (Agencia Tributaria española).
---

# CLAUDE.md — hacienda-cli

## Qué es este proyecto

CLI para interactuar con la sede electrónica de la AEAT (Agencia Tributaria española).
Soporta el Modelo 100 (IRPF / declaración de la renta), campaña 2025.

El CLI es deliberadamente mínimo: login, download, upload, validate. Toda la lógica
fiscal (generar el XML) es responsabilidad del agente AI, no del CLI.

## Comandos

```bash
hacienda login              # Abre Chromium, usuario se autentica, queda vivo
hacienda download 100       # Descarga datos fiscales (HTML) + borrador (PDF)
hacienda upload 100 f.xml   # Importa XML en EDFI de la AEAT
hacienda validate 100 f.xml # Valida XML contra XSD oficial (offline, xmllint)
hacienda info 100           # Muestra XSD, diccionario, URLs
```

## Archivos clave

- `src/index.ts` — CLI (commander). 4 subcomandos + info.
- `src/browser.ts` — Playwright: launch (headed) + connect (CDP).
- `data/Renta2025.xsd` — XSD oficial de la AEAT (ISO-8859-1, 811KB).
- `data/Renta2025-fixed.xsd` — Versión con regex corregidos para xmllint.
- `data/diccionarioXSD_2025.properties` — Diccionario campo→xpath→tipo→label (4009 líneas).
- `scripts/edfi-upload.mjs` — Script auxiliar para subir XML a EDFI y capturar errores ERES.

## Cómo generar un XML válido

El XML debe conformar a `Renta2025.xsd` y estar codificado en **ISO-8859-1**.

### Estructura raíz

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<Declaracion modelo="100" ejercicio="2025" periodo="0A" versionxsd="1.02">
  <Aux>
    <Idioma>E</Idioma>
    <VERSION>1.02</VERSION>
  </Aux>
  <DatosIdentificativos>
    <Declarante>...</Declarante>
    <Conyuge>...</Conyuge>           <!-- si casado -->
    <Hijos PH18="SI">...</Hijos>    <!-- si tiene hijos menores -->
  </DatosIdentificativos>
  <AsignacionTributaria>             <!-- opcional -->
    <FINESSOCIALES>SI</FINESSOCIALES>
  </AsignacionTributaria>
  <DatosEconomicos codigoCADeclaracion="12" TIPOTRIBUTACION="1">
    <TomaDatosAmpliada titular="2" nif="..." codigoCA="12">
      <RdtoTrabajo>...</RdtoTrabajo>
      <RdtoCapitalMobiliario>...</RdtoCapitalMobiliario>
      <Inmuebles>...</Inmuebles>
      <GPFondos>...</GPFondos>
      <GPAcciones>...</GPAcciones>
      <GPOtrosInmuebles>...</GPOtrosInmuebles>      <!-- ANTES de GPOtrosElementos -->
      <GPOtrosElementos>...</GPOtrosElementos>
      <CalculoImpuesto>...</CalculoImpuesto>         <!-- doble imposición -->
      <AnexoA>...</AnexoA>                           <!-- donativos -->
    </TomaDatosAmpliada>
    <TomaDatosAmpliada titular="3" nif="..." codigoCA="12">
      <!-- cónyuge: vacío si tributación individual -->
    </TomaDatosAmpliada>
    <Resultados>...</Resultados>
  </DatosEconomicos>
</Declaracion>
```

### Tipos de datos clave

- **tipo_logico**: `"0"` o `"1"` (NO "S"/"N")
- **tipo_SINO_Exclusivo**: `"SI"` o `"NO"` (para PH18, C_REV, etc.)
- **tipo_Fecha**: `dd/mm/yyyy` (string, max 10 chars)
- **tipo_ImpPositivo**: decimal >= 0, 2 decimales
- **tipo_ImpNegativo**: decimal (puede ser negativo), 2 decimales
- **tipo_Nif**: 9 caracteres alfanuméricos
- **tipo_ApeNom**: mayúsculas + acentos, max 80 chars

### Valores clave para enums

- **EstadoCivil**: 1=soltero, 2=casado, 3=viudo, 4=separado/divorciado
- **Sexo**: H=hombre, M=mujer
- **Titular**: 2=declarante, 3=cónyuge, 4-7=hijos
- **codigoCA**: 01=Andalucía, 02=Aragón, 03=Asturias, 04=Baleares, 05=Canarias, 06=Cantabria, 07=C. La Mancha, 08=C. León, 09=Cataluña, 10=Extremadura, 11=Galicia, 12=Madrid, 13=Murcia, 16=La Rioja, 17=C. Valenciana, 18=Ceuta, 19=Melilla, 20=No residente
- **TIPOTRIBUTACION**: 1=individual, 2=conjunta
- **VINCUDLG** (hijos): A=ambos progenitores, B=solo declarante, C=solo cónyuge

### Secciones dentro de TomaDatosAmpliada

El orden de elementos es estricto (xs:sequence). Orden correcto:

1. `RdtoTrabajo` — Rendimientos del trabajo
2. `RdtoCapitalMobiliario` — Capital mobiliario (intereses, dividendos)
3. `Inmuebles` — Inmuebles (vivienda habitual, a disposición, arrendados)
4. `GPFondos` — Fondos de inversión (IIC no cotizadas)
5. `GPFondosCoti` — ETFs / SICAV cotizadas
6. `GPAcciones` — Acciones cotizadas (negociadas en mercados)
7. `GPDerechos` — Derechos de suscripción
8. `GPOtrosCriptomonedas` — Criptomonedas
9. `GPOtrosInmuebles` — Venta de inmuebles
10. `GPOtrosElementos` — Acciones NO negociadas, private equity
11. `GPOtrasGanancias` — Otras ganancias patrimoniales
12. `CalculoImpuesto` — Doble imposición internacional (DOBIMPINT)
13. `AnexoA` — Donativos, deducción vivienda, etc.

### Mapping de campos GPAcciones

```xml
<EntidadAccion>
  <G2B_DE>NOMBRE ENTIDAD</G2B_DE>           <!-- nombre, NO ISIN -->
  <G2B_A valor="57378.19">                   <!-- valor = total transmisión -->
    <ENTIDAD>NOMBRE ENTIDAD</ENTIDAD>
    <TRANSACCION>
      <IT>57378.19</IT>                      <!-- valor transmisión -->
      <IA>30513.83</IA>                      <!-- valor adquisición -->
    </TRANSACCION>
  </G2B_A>
  <G2B_B>30513.83</G2B_B>                   <!-- total adquisición -->
  <G2B_C>26864.36</G2B_C>                   <!-- ganancia (0 si pérdida) -->
  <G2B_D>26864.36</G2B_D>                   <!-- ganancia reducida -->
  <!-- O si hay pérdida: -->
  <G2B_E>2693.36</G2B_E>                    <!-- pérdida -->

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jatorre/hacienda-cli](https://github.com/jatorre/hacienda-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
