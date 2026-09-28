---
trigger: always_on
description: Lee primero [README.md](README.md): qué es, cómo se arranca y cómo está montado.
---

# AS Video Studio — lo que hay que saber antes de tocarlo

Lee primero [README.md](README.md): qué es, cómo se arranca y cómo está montado.
Y si vienes a retomar sin contexto, [docs/RETOMAR.md](docs/RETOMAR.md): el
estado al cerrar, qué está desplegado y **los fallos que se encontraron con su
causa** — el sitio donde mirar antes de volver a tocar algo.

Esto es lo otro — lo que no se deduce leyendo el código y cuesta dinero o una
tarde averiguar.

---

## LO PRIMERO: aquí se paga por generar

Una tanda de imágenes de un vídeo de cuatro minutos son ~126 imágenes y ~4,4 $.
No es un detalle de producto: es la razón de la mitad de las decisiones de este
repo, y de estas tres reglas.

**1. No escribas params «por defecto» al abrir una pantalla.** La firma de un
paso se calcula sobre los params GUARDADOS. Escribir un valor que ese proyecto
nunca tuvo mueve la firma, deja obsoleto el paso y todo lo que cuelga, y la
pantalla ofrece regenerar el vídeo entero sin que nadie haya pedido nada. Las
pantallas editan una copia y solo guardan cuando alguien toca algo.

**2. Las tandas de imágenes se lanzan DE UNA EN UNA.** Dos a la vez tardan el
doble por imagen y pierden el registro del gasto.

**3. Un guardián calibrado sobre un fallo aprende a dar por bueno ese fallo.**
`p6_assets._planos_repetidos` tumba la tanda si dos planos acaban con la MISMA
imagen. Tiene dos excepciones —la cartela y la continuación de un plano largo—
y las dos están explicadas en su docstring. Si añades una tercera, comprueba
antes que lo que la justifica no es un bug.

---

## El grafo, y por qué no se toca

```
ingesta → brief → guion → voz → revision_audio → assets → callouts → render
```

Los ids **no se renombran** y **no se quita ninguno**, ni el que parezca que no
hace nada. `brief` es una cuenta determinista de dos segundos que no llama a
ningún modelo, y aun así sigue en el grafo: su firma encadena la del guion, así
que sacarlo dejaría obsoleto el guion de todos los proyectos guardados, en
cascada hasta el render. Mover su PANTALLA no cuesta nada; mover el PASO es una
migración.

### Lo que hay que saber de `nucleo/estado.py`

- **El manifiesto de cada versión vive fuera de `estado.json`**, en
  `pasos/<paso>/_versiones/v<N>.json`. Antes iba dentro y crecía como versiones
  × unidades: un vídeo de 300 planos llegó a 320 MB de `estado.json` y 9 s por
  petición. `Estado.manifiesto_version` **lanza** si el fichero falta o trae
  menos unidades de las que dice la versión: devolver `{}` vaciaría el mapa vivo
  y la pantalla ofrecería regenerar el vídeo entero. Se sigue leyendo el formato
  antiguo, y eso es permanente: un proyecto que llegue de fuera funciona sin
  migrar.
- **`completar()` escribe el manifiesto ANTES de tocar `datos`.** No es estilo:
  al revés, un fallo dejaría en disco `activa: N` con `versiones` sin la N, y
  «Revertir» enseñaría las imágenes de la versión anterior como si fueran las
  recién pagadas.
- **La carpeta de versiones va FUERA de `v<N>/`**: `sembrar_trabajo` copia la
  carpeta de la versión activa a `trabajo/` y `_recoger_trabajo` la vuelca en la
  SIGUIENTE, así que un manifiesto dentro de `v21` acabaría dentro de `v22` con
  pinta de correcto.
- **No caches el manifiesto en el documento.** `_guardar` filtra las claves con
  guion bajo SOLO en la raíz, así que una cache anidada se escribiría dentro de
  `estado.json` y el fichero recuperaría sus 320 MB en silencio.

---

## Copiar un proyecto o un estilo a otra máquina

**No basta con copiar la carpeta.** Un proyecto guarda rutas absolutas en tres
sitios: las salidas de cada unidad, los manifiestos de cada versión y las
referencias de estilo — y ese último entra en la firma del paso. Copiar y ya
deja `assets`, `callouts` y `render` en obsoleto: la pantalla ofrece regenerar
todos los planos ya pagados.

Hay que **mudar las rutas y volver a sellar las firmas**, y el sellado tiene dos
trampas:

1. **No se re-sella todo, solo lo que la mudanza movió.** `guion` guarda a
   propósito una `firma` que no cuadra con la calculada (cuando alguien corrige
   el guion a mano manda `firma_propia` y la otra se queda atrás). «Arreglarla»
   mueve el sello de salida del guion, que entra en la firma de la voz, y la
   cascada deja obsoleto todo lo de abajo. La regla es comparar contra la firma
   **calculada ANTES de mudar**.
2. **Los manifiestos del histórico también llevan firmas de unidad.** Sin
   re-sellarlos el estado vivo queda perfecto y cada «Revertir» enseña todas las
   unidades en obsoleto.

Y la comprobación es la mitad del trabajo: hay que contar **cuántos de los
ficheros que el estado dice haber producido están de verdad en disco**, en las
dos máquinas, y comparar. Una mudanza a medias no da ningún error al abrir el
proyecto; se ve al pedir una miniatura.

---

## La interfaz: una pantalla, y sin dependencias

`web/app.js` son ~10.000 líneas de JavaScript a pelo — sin framework y sin
`npm install`. Se sirve con `?v=<sello>` calculado del propio fichero, así que
**no hay ningún número que subir a mano** al cambiarlo.

**HAY UNA SOLA PANTALLA.** Hubo dos modos (uno guiado y uno desglosado en
pestañas) y aquí solo está el guiado. Si te encuentras algo que parece del otro

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NeverBlink/as-video-studio](https://github.com/NeverBlink/as-video-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-27 -->
