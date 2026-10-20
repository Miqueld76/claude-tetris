# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Proyecto

Tetris en JavaScript vanilla + HTML5 Canvas. Sin `package.json`, sin build, sin linter, sin tests. La documentación del proyecto (README) está en español.

## Ejecutar

- Abrir `index.html` directamente (`open index.html`), o servir estático: `python3 -m http.server 8000`.
- Para verificar cambios hay que probarlos en el navegador; no existe suite de tests.

## Arquitectura

Todo el juego vive en `game.js` (un único script, `'use strict'`, estado en variables globales de módulo). `index.html` aporta los elementos por id que `game.js` busca al cargar (`board`, `next-canvas`, `score`, `lines`, `level`, `overlay`, `overlay-title`, `overlay-score`, `restart-btn`); si se renombra un id hay que cambiarlo en ambos archivos.

Puntos que requieren entender varias partes a la vez:

- **Ciclo de vida de pieza**: `lockPiece()` = `merge()` → `clearLines()` → `spawn()`. `spawn()` promueve `next` a `current`, genera nueva `next`, y llama a `endGame()` si la pieza recién aparecida ya colisiona. Tanto `loop` (caída por gravedad), `softDrop` como `hardDrop` pasan por `lockPiece()`.
- **Bucle**: `loop(ts)` acumula `dropAccum` y baja una fila al superar `dropInterval`; el ID de `requestAnimationFrame` se guarda en `animId` y se cancela en pausa, game over e `init()` (reinicio). Al reanudar tras pausa se reinicia `lastTime` para evitar un salto de dt.
- **Dificultad**: `clearLines()` recalcula `level` (cada 10 líneas) y `dropInterval = max(100, 1000 - (level-1)*90)`; `init()` fija `dropInterval = 1000`, así que ambos valores deben mantenerse coherentes.
- **Piezas**: `PIECES[i]` son matrices cuyas celdas contienen el índice de color (1–7) que indexa `COLORS`; el tablero guarda ese mismo índice (0 = vacío). Rotación = `rotateCW` (transpuesta + reverso) con wall kicks simples `[0,-1,1,-2,2]` en `tryRotate`.
- **Tamaño del canvas**: `COLS`/`ROWS`/`BLOCK` en `game.js` deben coincidir con `width`/`height` del `<canvas id="board">` en `index.html` (300×600). `drawNext` usa su propio `NB = 30` sobre un canvas de 4×4 celdas.
- **Puntuación**: `LINE_SCORES × level` al limpiar líneas; hard drop +2 por celda, soft drop +1 por fila (la gravedad automática no puntúa).
