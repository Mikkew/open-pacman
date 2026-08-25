# SPEC 03 — Power Pellets y modo asustado

> **Estado:** Implementado
> **Depende de:** SPEC 01, SPEC 02
> **Fecha:** 2026-08-25
> **Objetivo:** Añadir power pellets que permiten a Pac-Man comer fantasmas durante un modo asustado temporal de 6 segundos.

## Alcance

**Incluido:**

- 4 power pellets en las celdas `(1,3)`, `(26,3)`, `(1,23)`, `(26,23)`, con nueva codificación `'o'` → valor `4` en el grid.
- Comer un power pellet suma 50 puntos, decrementa `dotsRemaining` y activa el modo asustado (6s).
- Al activarse, los fantasmas fuera de la pen invierten dirección, se vuelven azules, se mueven de forma aleatoria y más lenta (`FRIGHTENED_SPEED = 0.05`).
- Comer fantasmas asustados suma 200, 400, 800, 1600 (combo que se reinicia con cada pellet).
- Un fantasma comido pasa a modo "ojos": camina de vuelta a la pen (`PEN_CENTER`) y renace como fantasma normal.
- Los fantasmas dentro de la pen no se ven afectados por el modo asustado.
- Render: power pellets grandes, fantasmas asustados azules, "ojos" dibujados sin cuerpo.

**Fuera de alcance (para specs futuros):**

- Parpadeo de fantasmas antes de revertir (color fijo, decisión tomada).
- Ciclos chase/scatter.
- Velocidades diferenciadas por fantasma.
- Power pellets con animación de parpadeo.

## Modelo de datos

```js
// maze.js — nueva codificación 'o' -> 4 en parseTile()
// Filas 3 y 23 pasan a usar 'o' en cols 1 y 26:
//   fila 3:  '#o####.#####.##.#####.####o#'
//   fila 23: '#o..##................##..o#'
const PEN_CENTER = { x: 13, y: 14 };   // interior de la pen (destino de los ojos)

// game.js — constantes nuevas
const POWER_PELLET_SCORE = 50;
const FRIGHTENED_DURATION = 6000;      // ms
const FRIGHTENED_SPEED = 0.05;         // 1/20 celda/frame (debe alinear a entero)
const GHOST_EAT_SCORES = [200, 400, 800, 1600];

// game.js — estado global
game: { ..., frightenedMs: 0, ghostEatCombo: 0 }

// game.js — cada ghost gana un campo
ghost: { ..., inPen: true, releaseTime: i*1500, mode: 'chase' }
// mode: 'chase' | 'frightened' | 'eyes'
```

## Plan de implementación

1. **`maze.js` — codificar power pellets.** `parseTile()` mapea `'o'` a `4`. Reemplazar las filas 3 y 23 de `MAZE_STR`. Añadir `PEN_CENTER` y exportarlo con `window.PEN_CENTER`.
2. **`game.js` — constantes y estado.** Añadir `POWER_PELLET_SCORE`, `FRIGHTENED_DURATION`, `FRIGHTENED_SPEED`, `GHOST_EAT_SCORES`. En `createGame()`: `frightenedMs: 0`, `ghostEatCombo: 0`, y `mode: 'chase'` en cada ghost. Contar `dots` incluyendo `v === 4`.
3. **`game.js` — activar/finalizar asustado.** Añadir `startFrightened(game)` (fija `frightenedMs`, resetea `ghostEatCombo`, invierte dirección y pone `mode='frightened'` a los fantasmas fuera de pen que no sean ojos). En `update()` decrementar `frightenedMs` y, al llegar a 0, pasar los `frightened` a `chase`.
4. **`game.js` — comer pellet.** En `movePacman()`, si `grid[y][x] === 4`: poner 0, `score += 50`, `dotsRemaining--`, `startFrightened(game)`.
5. **`game.js` — movimiento de fantasmas.** En `moveGhost()`: si `mode==='eyes'` apuntar a `PEN_CENTER` y renacer (`mode='chase'` + `releaseGhost()`) al llegar; si `mode==='frightened'` elegir dirección aleatoria y usar `FRIGHTENED_SPEED`; si `chase` usar la IA por `kind`. `releaseGhost()` fija `mode='chase'`.
6. **`game.js` — colisiones.** En `update()`: ignorar ojos; si `mode==='frightened'` comer (sumar `GHOST_EAT_SCORES[combo]`, `combo++`, `mode='eyes'`); si no, morir como ahora. `resetPositions()` limpia `frightenedMs`, `ghostEatCombo` y `mode`.
7. **`render.js` — dibujo.** Añadir `drawPowerPellets()` (círculo grande en celdas `4`), `FRIGHTENED_COLOR` azul, `drawEyes()` (solo ojos). En `draw()` elegir color/forma según `mode`.
8. **Verificación manual** contra los criterios de aceptación.

## Criterios de aceptación

- [ ] La partida muestra 4 power pellets grandes en `(1,3)`, `(26,3)`, `(1,23)`, `(26,23)`.
- [ ] Comer un power pellet suma 50 puntos y decrementa `dotsRemaining`.
- [ ] Al comerlo, los fantasmas fuera de la pen se vuelven azules, invierten dirección y se mueven aleatorio y más lento.
- [ ] Durante el modo asustado, Pac-Man come fantasmas sumando 200, luego 400, 800 y 1600.
- [ ] Un fantasma comido se convierte en ojos, camina de vuelta a la pen y renace como fantasma normal.
- [ ] El modo asustado dura 6 segundos; al terminar, los fantasmas vuelven a su IA de persecución.
- [ ] Los fantasmas dentro de la pen no se vuelven azules ni se ven afectados.
- [ ] Comer otro power pellet durante el asustado reinicia el temporizador a 6s y el combo a 200.
- [ ] Los power pellets cuentan para `dotsRemaining`: comer los 4 + todos los dots gana la partida.
- [ ] Sin errores en consola al cargar o jugar.

## Decisiones tomadas y descartadas

- **Sí:** Modo asustado de 6s, inversión de dirección y movimiento aleatorio+lento (clásico).
- **Sí:** Fantasma comido → ojos → caminan a `PEN_CENTER` → renacen vía `releaseGhost()` (reutiliza el teletransporte de salida del SPEC 02).
- **Sí:** Puntuación 200/400/800/1600 con combo reiniciado por cada pellet nuevo.
- **Sí:** Power pellet vale 50 puntos y cuenta para `dotsRemaining`.
- **Sí:** `mode` por fantasma (`chase`/`frightened`/`eyes`) en lugar de un flag global, para que los fantasmas que renacen o salen de la pen durante el asustado sean normales (fiel al clásico).
- **Sí:** Codificación `'o'` → `4` en `MAZE_STR`; `FRIGHTENED_SPEED = 0.05` (1/20) para que las posiciones sigan alineando a celda entera.
- **No:** Parpadeo de fantasmas (color fijo, pedido).
- **No:** Power pellet con animación de parpadeo.
- **No:** Asustado afectando a fantasmas en pen.
- **No:** Ciclos chase/scatter ni velocidades distintas por fantasma.

## Riesgos

| Riesgo | Mitigación |
|--------|------------|
| Los ojos no encuentran camino a la pen | `ghostOptions()` permite revertir si no hay otra opción; `pickBestDir()` siempre devuelve una dirección válida. |
| Velocidad asustada que no alinea a celda | `FRIGHTENED_SPEED = 0.05` = 1/20; alinea igual que `GHOST_SPEED` (1/10). |
| Invertir dirección a mitad de celda produce un pequeño salto visual | Aceptable: el fantasma retrocede hacia la celda entera de la que vino, que es transitable. |
| Renacimiento sobre Pac-Man en `GHOST_EXIT` | `GHOST_EXIT (13,11)` está lejos del inicio `(13,23)`; liberación escalonada (SPEC 02). |

## Qué **no** está en este spec

- Parpadeo de fantasmas antes de revertir.
- Ciclos chase/scatter.
- Velocidades diferenciadas por fantasma.
- Animación de parpadeo de los power pellets.
