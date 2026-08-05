---
trigger: always_on
description: Un **curso-repositorio** para aprender a construir un LLM desde cero programando en
---

# CLAUDE.md — instrucciones para sesiones de Claude Code en este repo

## Qué es esto

Un **curso-repositorio** para aprender a construir un LLM desde cero programando en
PyTorch. No es una librería ni un tutorial para leer: el usuario abre VSCode, lee la
teoría, implementa funciones marcadas con `NotImplementedError` y ejecuta tests hasta que
pasan. El objetivo final es un GPT de **8.933.440 parámetros** entrenado por él sobre
TinyStories hasta que genere historias cortas coherentes en inglés.

## EL PUNTO MÁS IMPORTANTE: a quién le hablas

El usuario es **ingeniero de software con experiencia** (Python, git, CLI, arquitectura)
pero su **base en LLMs y machine learning es baja**, y el curso está pensado para gente en
esa misma situación. Separa las dos cosas o te equivocarás en una de las dos direcciones:

- **No le expliques programación.** Qué es un bucle, qué es una clase, qué hace `dict.get`.
  Eso le hace perder el tiempo.
- **Sí explícale todo lo de ML desde cero.** Qué es un logit, por qué se normaliza, qué
  significa que un gradiente "se vaya a cero". Nada se da por sabido.

La primera versión de este repo se escribió demasiado técnica y hubo que reescribirla. El
brief original pedía teoría "densa, sin analogías, 400-900 palabras"; esa instrucción quedó
**derogada** el 2026-07-30 tras ver el material. Lo que manda es lo de abajo.

### Estructura obligatoria de cada explicación: intuición → ejemplo → fórmula

Cada concepto de ML entra **tres veces y en este orden**:

1. **Qué problema resuelve**, en lenguaje llano. Sin fórmulas, sin jerga sin definir.
2. **Un ejemplo con números pequeños** que el lector pueda seguir a mano. Matrices de 2×3,
   tres palabras, cuatro conteos. Números concretos, no símbolos.
3. **La fórmula formal**, conectada explícitamente con el ejemplo anterior.

La matemática **no se elimina ni se esconde**: deja de ser lo primero que se lee. Un
`TEORIA.md` que abre con `C_token ≈ 6N + 12·n_layers·T·d_model` está mal escrito aunque la
fórmula sea correcta.

### Estructura obligatoria de cada docstring de ejercicio

Es la misma idea aplicada al código, y se estableció el 2026-07-31 después de que el usuario
dijera "yo leo esto y no sé qué tengo que hacer". El diagnóstico: los docstrings explicaban
**qué** era la función y **por qué** hacía falta, pero nunca **qué teclear**.

El orden es fijo, y **`QUÉ TIENES QUE ESCRIBIR` va primero, antes que cualquier teoría**:

```
<Una frase: qué hace la función>

QUÉ TIENES QUE ESCRIBIR
-----------------------
<Pasos numerados con el código concreto a teclear, indentado. No pseudocódigo:
 las líneas reales, con los nombres reales de las variables.>

QUÉ TIENE QUE SALIR / COMPRUÉBALO CON...
----------------------------------------
<Números concretos que el lector pueda verificar a mano>

POR QUÉ / DE DÓNDE SALE ESA FÓRMULA
-----------------------------------
<La intuición, ahora que ya sabe qué está escribiendo>

CUIDADO CON... / LOS ERRORES QUE HAY QUE EVITAR
-----------------------------------------------
<Las trampas, y sobre todo las silenciosas: las que no dan error y degradan el resultado>

Args: / Returns: / Raises:
```

Los 62 docstrings siguen este formato. Al añadir uno nuevo, el barrido de comprobación es:

```python
# recorre CURRICULUM y verifica que cada símbolo tiene la sección
"QUÉ TIENES QUE ESCRIBIR" in ast.get_docstring(nodo)
```

Dos reglas que salieron de escribirlos:

- **El código de los pasos tiene que compilar con lo que el alumno tiene importado.** Si tu
  paso usa `math.sqrt` y `ejercicios.py` no importa `math`, el paso está mal. Pasó de verdad
  en el módulo 12 (se cambió a `** 0.5`) y en el 01 (faltaba `import time` en el fichero).
- **Los números de ejemplo se miden, no se estiman.** Los del módulo 17 se inventaron
  plausibles y estaban mal por un factor de 5; hay que ejecutar la referencia y copiar.

## Reglas de escritura, innegociables

- **Toda la prosa en español**: teoría, docstrings, comentarios, mensajes de la CLI,
  nombres de los tests. **Los identificadores del código en inglés**: `causal_mask`,
  `n_heads`, `class MultiHeadAttention`. Esta mezcla es deliberada.
- **La prosa de los `.py` va sin tildes ni `ñ`**: `tamanyo`, `ensenyar`, `pequenyo`. Dos
  excepciones, y solo dos: los **títulos de sección en mayúsculas** de los docstrings sí las
  llevan (`QUÉ TIENES QUE ESCRIBIR`, `POR QUÉ ESA FÓRMULA`) porque son los puntos de anclaje
  al leer en diagonal, y los emoji de la CLI, que viven en `progress.py`. Los `.md` llevan
  tildes normales en todas partes.
- **Cada `TEORIA.md` abre con `## Por qué importa este módulo`**, ANTES de cualquier
  concepto: qué problema resuelve, qué sabrás al terminar, y cuánto cuesta. Hay un test que
  lo verifica. Alguien que no sabe de LLMs no puede juzgar si merece la pena leer cuatro
  horas sobre atención si no le dices primero que es LA pieza que separa un modelo mediocre
  de ChatGPT.
- **Mínimo 900 palabras por `TEORIA.md`, sin techo.** El límite superior se quitó el
  2026-07-31: cada concepto se explica lo que haga falta, con ejemplos concretos.
- **Cada `SOLUCION.md` termina con `## El código completo`**: la implementación entera de
  todos los ejercicios del módulo, copiable. Hay un test que lo verifica. El brief original

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [roberottt/llm-creator-studio](https://github.com/roberottt/llm-creator-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-05 -->
