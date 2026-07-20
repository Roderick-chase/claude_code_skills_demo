# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

- `npm run dev` — start the Vite dev server
- `npm run build` — production build to `dist/`
- `npm run preview` — serve the production build locally

There is no test suite or linter configured.

## Architecture

Single-page tech landing page ("Particle Nexus") built with Vue 3 + Vite. The only runtime dependency is `vue` — no router, no state management, no third-party particle library.

Dependency chain: `index.html` → `src/main.js` → `src/App.vue` → `src/components/ParticleCanvas.vue`.

- `src/App.vue` — page layout (HUD header with stats, hero section, footer, decorative glows/scanline/corners). Receives particle count and FPS from ParticleCanvas via a `stats` event.
- `src/components/ParticleCanvas.vue` — the particle engine, written directly against the Canvas 2D API with a `requestAnimationFrame` loop. Tunable via props (`density`, `linkDistance`, `mouseRadius`); App currently uses the defaults.

## Key design constraints

- Particle state (`particles`, `bursts`, mouse position) intentionally lives in plain module-level variables inside ParticleCanvas, NOT in Vue reactive refs — reactivity per animation frame would be a performance problem. Keep it that way when modifying the engine.
- The foreground `.overlay` in App.vue uses `pointer-events: none` so mouse events pass through to the canvas (particle interaction: mouse attracts particles, click triggers a burst). Interactive elements like buttons must individually restore `pointer-events: auto`.
- The engine cleans up in `onBeforeUnmount` (cancels rAF, removes listeners); any new listeners or timers added to ParticleCanvas must be cleaned up there too.
