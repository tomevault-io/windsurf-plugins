---
trigger: always_on
description: `README.md` cuenta qué es el proyecto y por qué. Acá está lo que hace falta para
---

# FlyBrain — contexto para trabajar en este repo

`README.md` cuenta qué es el proyecto y por qué. Acá está lo que hace falta para
**tocar el código sin romperlo**: los invariantes, qué vive dónde, y qué se
intentó ya.

## Estado

El motor está terminado y en verde. La mosca corre en `fly/` (LIF sobre las
164.506 neuronas del conectoma) y maneja **dos** acciones del boss: la esquiva,
por la fibra gigante, y el movimiento, leyendo las motoneuronas de las seis patas
(`fly/patas.py`). El tacto de las patas la aparta de las paredes (pasa su
experimento y su control cruzado); DNa02 no sobrevive al LIF, así que lejos de
las paredes el rumbo es ruido de la red.

Y **ataca**: embiste cuando la balanza de valencia de las MBON del cuerpo
pedunculado dice acercarse (`fly/ataque.py`), con el olor del rival —la feromona
cVA— como entrada. La dopamina entra de verdad (PAM al acertar, PPL1 al recibir
daño) y mueve esa balanza, pero **todavía no cambia la conducta**: el control sin
plasticidad hace lo mismo. Esa mosca corre con ruido de fondo 1,5 y no 2,0,
porque con 2,0 la red se enciende sola y cierra el olfato; el precio es que la
esquiva pierde precisión. Todo medido en `fly/README.md`.

## El invariante del que cuelga todo

**El motor de combate es una función pura.** Sin I/O, sin estado global, sin
aleatoriedad fuera del `Rng` del `World`, paso fijo de 1/60. Sin excepciones.

Esto no es purismo: es lo que hace que una pelea grabada contra la mosca en una
GPU alquilada se reproduzca **bit a bit** en el navegador. Si diverge, el video
no muestra lo que pasó y el proyecto entero pierde el sentido.

Tres reglas concretas que salen de ahí, y que ya costaron caro cuando se
ignoraron:

1. **`libm`, nunca los métodos de `f32`.** `sin`, `cos`, `atan2` y `exp` salen de
   la libm de cada plataforma y **no coinciden entre wasm y x86**. Suma, resta,
   multiplicación, división y `sqrt` sí son exactas por IEEE 754 y no dan
   problema. `Vec2::angle` y `Vec2::from_angle` ya lo respetan: usalas.
2. **Ninguna regla de juego fuera de Rust.** Si una decisión de gameplay vive en
   TypeScript o en Python, hay dos implementaciones que pueden divergir. El
   navegador lee el buffer plano del wasm y dibuja; nada más.
3. **El RNG es propio (PCG32 en `rng.rs`), no el crate `rand`.** Su algoritmo
   puede cambiar entre versiones y eso rompería toda grabación vieja.

Si `el_hash_del_mundo_final_no_cambio` falla, **no actualices el hash sin
entender el diff**. Ese test es lo único que separa un cambio de balance de una
divergencia de plataforma.

## Los chequeos

Ninguno puede quedar en rojo. Son cuatro y corren en segundos:

```bash
cargo test                        # 112 tests
./scripts/wasm-determinism.sh     # nativo y wasm dan el mismo hash
node scripts/wasm-smoke.mjs       # el binding del navegador
cd web && npm run check           # la escena se dibuja y la pelea corre
```

## Qué vive dónde

| | hace | **no** hace |
|---|---|---|
| **Rust** (`engine/`) | Toda la simulación: física, colisiones, raycasts, daño, cooldowns, RNG. La única fuente de verdad. | Dibujar. Red. Archivos. Nada de I/O. |
| **TypeScript** (`web/`) | Pegamento fino: carga una pelea grabada, la avanza en el wasm, le da el estado a Three.js. | Cualquier regla de juego. |
| **Python** (`training/`) | Correr el conectoma contra el motor como entorno vectorizado. | Existir en producción. |

La prueba de que la frontera está bien: **borrar `training/` no debe afectar al
juego en absoluto**, y el motor debe correr una pelea entera sin que exista
TypeScript.

## El punto de integración

Hay exactamente uno, y es el contrato de `lib.rs`:

```rust
pub fn step(w: &mut World, input: PlayerInput, action: BossAction) -> StepEvents
```

**El motor no elige la acción del boss: la recibe.** Quien produzca un
`BossAction` por tick controla al boss.

Hoy no la produce nadie. Las dos vías construidas:

- **Python** (`engine/src/python.rs`): `VecEnv` toma `(n, 3)` uint8 y devuelve
  `(n, 43)` float32 + recompensa + done. Es donde entra el conectoma.
- **Navegador** (`engine/src/wasm.rs`): `load_log` + `step_log` reproducen una
  pelea grabada tick a tick. El log lo expande el motor, **no JavaScript**: el
  formato lo define `log.rs` y una segunda implementación en TS rompería la
  reproducibilidad.

`wasm.rs` también expone `step_tick`, que acepta la acción del boss desde fuera.
Es la puerta para el modo en vivo (opción C del README) cuando llegue.

### La API a Python es batcheada, y tiene que seguir siéndolo

Un `step()` por llamada hace que el overhead de FFI se coma la ventaja de Rust
entera y termine más lento que Node. Una llamada avanza N arenas en paralelo con
rayon y devuelve arrays contiguos que numpy lee sin copiar. Es la diferencia
entre ~50k y ~1M pasos por segundo. `training/env_smoke.py` mide esto y falla por
debajo de 200k.

## De dónde viene esto, y qué se quitó

Fork de EPOCH, un boss colectivo entrenado por RL contra jugadores reales. **Se
quitó todo lo que era de aquel proyecto:**

- `brain.rs` (2293 líneas): el bandit contextual. Aprendía qué herramienta usar
  en qué contexto, por contadores. Reemplazado por el conectoma.
- `replay.rs` (1123): el verificador anti-trampas. Re-simulaba el log del cliente

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Jhongdlp/FlyBrain](https://github.com/Jhongdlp/FlyBrain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
