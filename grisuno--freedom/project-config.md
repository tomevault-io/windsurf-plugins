---
trigger: always_on
description: > **Misión:** un navegador web construido desde cero en **C puro**, diseñado como respuesta
---

# Freedom — Navegador Seguro por Defecto

> **Misión:** un navegador web construido desde cero en **C puro**, diseñado como respuesta
> directa a la vigilancia corporativa (modelos tipo Brave–Palantir). Cero telemetría, cero
> backdoors, cero confianza implícita. Lo que no se puede auditar, no entra.

Este archivo es el contrato de trabajo para cualquier agente (humano o IA) que toque este
repositorio. **Estas reglas anulan comportamientos por defecto.**

---

## 1. Los seis principios inquebrantables

1. **Zero Trust** — Ningún componente confía en otro. El renderizador no confía en la red; el
   motor JS no confía en el DOM; nada confía en el contenido remoto. Aislamiento por límites
   estrictos de memoria y, donde el SO lo permita, `seccomp-bpf` (Linux), `landlock`,
   `pledge`/`unveil` (OpenBSD).
2. **Zero Knowledge** — El navegador no sabe del usuario más de lo estrictamente necesario para
   renderizar. Sin historial en claro, sin fingerprinting pasivo, sin fugas de IP (WebRTC
   deshabilitado por defecto).
3. **Privacy by Default** — Bloqueo total de terceros a nivel del motor de red. Sin telemetría
   ni siquiera "anónima" u "opt-out". Integración opcional con Tor/I2P a nivel de socket.
4. **Secure by Default** — La configuración insegura **no debe ser representable** en la API.
   El camino por defecto es siempre el seguro. Fallar cerrado: si una garantía no se puede
   verificar, se rechaza la operación.
5. **Post-Quantum by Default** — TLS 1.3 mínimo. Intercambio de claves **híbrido** (clásico +
   ML-KEM) para neutralizar *Harvest-Now, Decrypt-Later*. Nunca PQ puro (si ML-KEM cae, el
   componente clásico debe resistir); nunca clásico puro.
6. **Agent-Safe & Agent-Friendly** — Seguro para el usuario **y** para el agente de IA que lo
   opere, en ambas direcciones: el contenido remoto es hostil también para el agente (inyección
   de prompts), así que se le entrega siempre como **dato con procedencia, nunca como
   instrucción**, y sin acción implícita; y el navegador es manejable por un agente (salidas
   deterministas, con códigos de estado, sin estado oculto, *headless*). El agente opera dentro
   de los mismos sandboxes que el usuario. Contrato completo en `spec/agent-safety.md`.

**Doctrina anti-vigilancia:** no se permite ninguna cadena de texto, dependencia, endpoint ni
comentario que apunte a servicios de terceros no esenciales. Cada dependencia se justifica por
reducción de superficie de ataque, no por conveniencia.

---

## 2. Restricciones de lenguaje y estilo

- **Solo C puro (C11).** Nada de C++, Rust ni dependencias ocultas. El header rechaza C++ con
  `#error`.
- **Identificadores y strings en inglés.** La documentación (`spec/`, este archivo) puede estar
  en español; el código, no.
- Sin emojis en el código. Comentarios solo cuando explican un *porqué* no evidente. Los headers
  llevan documentación de contrato.
- Nombres con prefijo de módulo (`sf_` para `secure_fetch`, etc.). Sin estado global mutable;
  todo reentrante. Cada asignación tiene un único dueño y un único liberador idempotente.

---

## 3. Metodología: SDD + TDD estricto + BDD Given-When-Then

Para cada módulo el ciclo es inviolable y **en este orden**:

1. **Spec** — `spec/<modulo>.md`: entradas, salidas, tabla de errores, garantías de seguridad y
   qué queda fuera de alcance, en Dado-Cuando-Entonces.
2. **Test (rojo)** — `tests/test_<modulo>.c` con CMocka (ATDD). Debe **fallar de verdad**:
   verificá el rojo revirtiendo el fix, no lo supongas. Un build que falla por `-Werror` deja el
   binario viejo en su sitio y el test "pasa" — eso no es rojo, es un experimento inválido.
3. **Code (verde)** — `src/<modulo>.c`, el código mínimo para pasar. La I/O del **lado confiable**
   (orquestador / event loop, el que NO toca contenido hostil) debe ser **asíncrona** (`io_uring`
   cuando aplique). **Excepción inquebrantable:** `io_uring` está **PROHIBIDO dentro del worker
   confinado** (`tab`/`renderer`): es una **primitiva de bypass de seccomp** (sus `IORING_OP_*` no
   atraviesan el syscall entry que filtra el BPF → anularía allowlist, W^X y netns). El worker hace
   I/O **bloqueante** sobre sus dos pipes. `spec/os_sandbox.md` §13,
   `[[freedom-io-uring-forbidden-in-worker]]`.
4. **Refactor** — endurecer punteros y límites. **Modo boy scout, nunca fuera de scope:** código
   duplicado se unifica; deuda técnica y vulnerabilidades se extinguen sin perder funcionalidad;
   si algo se hace en 10 líneas en vez de 40 respetando DRY/SOLID, se hace.
   **Cláusula anti-monolito:** ningún archivo se vuelve monolito. Al rozar las **~2000 líneas** se
   **parte según contratos** (módulo = `spec/` + `include/` + `src/`). Si tu cambio empujaría un
   archivo más allá del umbral, **primero extraé**. **Deuda conocida:** `gui/browser_ui.c` ya
   excede el umbral (>12.000 líneas) — al tocarlo, la lógica nueva (sobre todo la pura) va a un
   módulo nuevo, no a engordarlo.
5. **Validación** — `make asan` (ASan+UBSan) limpio, `valgrind`, `cppcheck`, **más revisión visual
   del render**. La GUI necesita Wayland (no siempre disponible), así que se inspecciona headless:
   `./build/freedom --download-png=$SP/frame.png <URL-o-archivo.html>` y `Read` de la imagen.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [grisuno/FreeDom](https://github.com/grisuno/FreeDom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
