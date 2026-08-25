# SPEC 02 — Salida de los fantasmas de la pen

> **Estado:** Implementado
> **Depende de:** SPEC 01
> **Fecha:** 2026-08-25
> **Objetivo:** Corregir la salida de los fantasmas de la pen: al liberarse se teletransportan a la celda `(13, 11)`, fuera de la pen, con dirección `'left'` y desde ahí arranca su IA de persecución.

## Alcance

**Incluido:**

- Teletransportar al fantasma a la celda `(13, 11)` (justo arriba-izquierda de la puerta) en el instante en que se libera (`inPen` pasa a `false`).
- Dirección inicial `'left'` al aparecer fuera de la pen.
- Los 4 fantasmas salen a su `releaseTime` ya programado (0s, 1.5s, 3.0s, 4.5s), sin quedar atrapados dentro de la pen.
- Los fantasmas que aún no se liberan permanecen quietos dentro de la pen (sin cambios respecto a lo actual).

**Fuera de alcance (para specs futuros):**

- Animación de bouncing dentro de la pen mientras esperan la liberación.
- Recorrido físico de la salida (caminar hasta la puerta y cruzarla) en lugar de teletransporte.
- Retorno de los fantasmas a la pen al ser comidos.

## Modelo de datos

```js
// maze.js — nueva constante de salida
const GHOST_EXIT = { x: 13, y: 11 };

// game.js — helper de liberación
function releaseGhost( game, g ) {
  g.inPen = false;
  g.x = GHOST_EXIT.x;
  g.y = GHOST_EXIT.y;
  g.dir = 'left';
}
```

`GHOST_STARTS` no cambia: los fantasmas siguen arrancando dentro de la pen. Solo cambia dónde se colocan al liberarse.

## Plan de implementación

1. **Añadir `GHOST_EXIT` en `maze.js`.** Declarar `const GHOST_EXIT = { x: 13, y: 11 };` y exportarla con `window.GHOST_EXIT = GHOST_EXIT;`.
2. **Añadir `releaseGhost` en `game.js`.** Helper que marca `inPen = false`, coloca al fantasma en `GHOST_EXIT` y fija `dir = 'left'`.
3. **Usar `releaseGhost` en `update()`.** Sustituir la línea `if ( g.inPen && game.elapsedMs >= g.releaseTime ) g.inPen = false;` por una llamada a `releaseGhost( game, g )`.
4. **Verificar `resetPositions()`.** No requiere cambios: sigue restaurando a `GHOST_STARTS` con `inPen: true`, y la liberación vuelve a teletransportar.

## Criterios de aceptación

- [ ] Al alcanzar su `releaseTime`, cada fantasma aparece en `(13, 11)` con dirección `'left'` y comienza a moverse.
- [ ] Ningún fantasma queda atrapado dentro de la pen tras ser liberado.
- [ ] Los 4 fantasmas se liberan a sus tiempos programados (0s, 1.5s, 3.0s, 4.5s).
- [ ] Los fantasmas aún no liberados permanecen quietos dentro de la pen.
- [ ] Sin errores en consola al cargar o jugar.

## Decisiones tomadas y descartadas

- **Sí:** Teletransporte directo a `(13, 11)` al liberarse. Simple, predecible y cumple lo pedido.
- **Sí:** Celda de salida `(13, 11)`. Es la salida clásica sobre la puerta de la pen.
- **Sí:** Dirección inicial `'left'`. El fantasma arranca en movimiento sin depender de una decisión de IA en el primer frame.
- **No:** Recorrer físicamente la puerta antes de activar la IA. Requiere pathfinding dentro de la pen y es más complejo de lo necesario.
- **No:** Bouncing dentro de la pen. Ya declarado fuera de alcance en SPEC 01.

## Riesgos

| Riesgo | Mitigación |
|--------|------------|
| El teletransporte solapa con otro fantasma en `(13, 11)` | La liberación es escalonada (1.5s de margen); el juego ya permite superposición entre fantasmas. |
| Colisión inmediata con Pac-Man al aparecer | `(13, 11)` está lejos del inicio de Pac-Man `(13, 23)`; la probabilidad es mínima y la colisión se resuelve con las reglas ya existentes. |

## Qué **no** está en este spec

- Bouncing dentro de la pen.
- Recorrido físico de la salida por la puerta.
- Retorno a la pen al ser comidos.
