# SPEC 01 — Cuatro fantasmas con IA diferenciada

> **Estado:** Implementado
> **Depende de:** —
> **Fecha:** 2026-08-24
> **Objetivo:** Cuatro fantasmas con comportamientos de IA distintos y liberación escalonada desde la pen, replicando los patrones clásicos de Pac-Man.

## Alcance

**Incluido:**

- 4 fantasmas (`blinky`, `pinky`, `inky`, `clyde`) con comportamientos únicos.
- Blinky persigue directamente a Pac-Man (hereda el comportamiento actual de `hunter`).
- Pinky apunta 4 celdas adelante de Pac-Man en la dirección actual de Pac-Man (emboscada).
- Inky calcula su objetivo usando la fórmula clásica: tile 2 celdas adelante de Pac-Man, duplica el vector desde Blinky a ese tile, y apunta ahí.
- Clyde persigue directamente a >8 celdas de distancia de Pac-Man; a ≤8 celdas se mueve aleatoriamente.
- Liberación escalonada desde la pen: un fantasma nuevo cada 1.5 segundos desde el inicio de la partida.
- Los 4 fantasmas arrancan dentro de la pen, todos en la columna 14, filas 12 a 15.
- Colores ya definidos en `render.js`: rojo para blinky, cyan para pinky, rosa para inky, naranja para clyde.

**Fuera de alcance (para specs futuros):**

- Power pellets y modo asustado (frightened).
- Ciclos chase/scatter periódicos.
- Fantasmas regresando a la pen al ser comidos.
- Velocidades diferenciadas por fantasma.
- Animación de bouncing dentro de la pen mientras esperan la liberación.

## Modelo de datos

```js
// maze.js — GHOST_STARTS pasa de 2 a 4 entradas
const GHOST_STARTS = [
  { x: 14, y: 12, kind: 'blinky' },
  { x: 14, y: 13, kind: 'pinky' },
  { x: 14, y: 14, kind: 'inky' },
  { x: 14, y: 15, kind: 'clyde' },
];

// game.js — cada ghost gana dos campos
ghost: {
  ...,
  inPen: true,
  releaseTime: 0,
}

// game.js — el estado del juego gana un contador
game: {
  ...,
  elapsedMs: 0,
}
```

| Índice | Fantasma | releaseTime (ms) | Color   |
|--------|----------|-------------------|---------|
| 0      | blinky   | 0                 | rojo    |
| 1      | pinky    | 1500              | cyan    |
| 2      | inky     | 3000              | rosa    |
| 3      | clyde    | 4500              | naranja |

## Plan de implementación

1. **Actualizar `GHOST_STARTS` en `maze.js`.** Reemplazar las 2 entradas por las 4 con nombres nuevos.
2. **Extender el estado en `game.js`.** `createGame()` inicializa `elapsedMs: 0` y cada ghost con `inPen: true` + `releaseTime` según su índice. `resetPositions()` restaura estos campos.
3. **Lógica de liberación en `update()`.** Sumar ~16.67 ms por frame a `elapsedMs`. Ghosts con `inPen && elapsedMs >= releaseTime` se liberan. Ghosts en pen no se mueven ni colisionan.
4. **Implementar 4 comportamientos.** Reemplazar `decideGhost()` por `decideBlinky`, `decidePinky`, `decideInky`, `decideClyde`. `moveGhost()` solo decide si `!g.inPen`.
5. **Verificar colores.** Los 4 colores en `GHOST_COLORS` ya mapean correctamente por índice.

## Criterios de aceptación

- [x] La partida inicia con 4 fantasmas dentro de la pen.
- [x] Blinky sale inmediatamente y persigue a Pac-Man directamente.
- [x] Pinky se libera a los 1.5s y apunta 4 celdas adelante de Pac-Man.
- [x] Inky se libera a los 3.0s y usa la posición de Blinky para su objetivo.
- [x] Clyde se libera a los 4.5s: persigue a >8 celdas, aleatorio a ≤8.
- [x] Los fantasmas en pen no se mueven ni colisionan hasta ser liberados.
- [x] Cada fantasma usa un color distinto (rojo, cyan, rosa, naranja).
- [x] Sin errores en consola al cargar o jugar.

## Decisiones tomadas y descartadas

- **Sí:** 4 comportamientos clásicos. Reconocibles, documentados, cumplen el requisito.
- **Sí:** Liberación cada 1.5s. Margen para el jugador.
- **Sí:** Todos arrancan dentro de la pen. Blinky sale instantáneamente.
- **Sí:** Misma velocidad. Solo se pidieron comportamientos distintos.
- **Sí:** Inky depende de Blinky (`game.ghosts[0]`). Fiel al original.
- **No:** Ciclos chase/scatter, power pellets, retorno a la pen, bouncing, velocidades distintas.

## Riesgos

| Riesgo | Mitigación |
|--------|------------|
| Inky mal calculado si Blinky aún está en pen | Inky se libera 3s después de Blinky; Blinky ya salió. |
| Pinky apunta a celda inalcanzable (muro) | Retroceder hasta la última celda transitable en esa dirección. |
| Demasiada dificultad con 4 perseguidores | La liberación escalonada da margen. Ajustable sin cambiar arquitectura. |

## Qué **no** está en este spec

- Power pellets y modo asustado.
- Ciclos chase/scatter.
- Retorno a la pen al ser comidos.
- Velocidades distintas.
- Animación de bouncing.
