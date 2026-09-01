# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

@AGENTS.md

## Commands

```bash
npm run dev      # next dev
npm run build    # next build
npm start        # next start (requires a prior build)
npm run lint     # eslint (flat config, no args needed)
npx eslint app/page.tsx   # lint a single file
npx tsc --noEmit          # typecheck only (tsconfig has noEmit)
```

No test runner is configured yet — `package.json` has no `test` script and no testing dependency. If tests are added, record the single-test invocation here.

## Stack notes

- **Next.js 16 App Router + React 19.** The bundled docs in `node_modules/next/dist/docs/` (`01-app/`, `02-pages/`, `03-architecture/`) are the authority for this version's APIs — read them before writing routing/data-fetching code, per `AGENTS.md`.
- **Typed route props are generated, not imported.** `app/layout.tsx` uses the global `LayoutProps<"/">` type emitted into `.next/types/`; there is no import for it. Page components get `PageProps<"/route">` the same way. Types come from `next dev`/`next build`, so a stale `.next/` means missing route types.
- **Tailwind v4, CSS-first.** There is no `tailwind.config.*`. Theme tokens are declared in `app/globals.css` via `@import "tailwindcss"` + `@theme inline { ... }`, wired through PostCSS (`@tailwindcss/postcss`). Add design tokens there, not in a JS config.
- Path alias `@/*` maps to the repo root.

## Project state and where the design lives

`app/` is still the unmodified `create-next-app` scaffold (`page.tsx` is the starter template, `layout.tsx` metadata says "Create Next App"). The actual product — a retro-arcade portal where players play browser games and compete for high scores — exists only as a **static prototype** in `references/resources/resources/templates/`. Treat it as the design spec to port, not as code to import.

The prototype (`Arcade Vault.html`) loads React 18 UMD + Babel-standalone from a CDN and transpiles `.jsx` in the browser. Each file defines globals (`window.Nav`, `window.GAMES`, …) rather than using ES modules, so nothing there can be imported into the Next app as-is — components must be rewritten as real modules.

Prototype structure, and how it maps to work in `app/`:

| File | Component | Intended route |
| --- | --- | --- |
| `data.jsx` | `GAMES`, `CATS`, `seededScores()` | mock data source (8 games, deterministic LCG-seeded scoreboards) |
| `nav.jsx` | `Nav` | shared chrome (desktop links + mobile slide-over) |
| `biblioteca.jsx` | `Library`, `GameCard` | `/` — grid, search, category filter |
| `detalle.jsx` | `GameDetail` | `/games/[id]` |
| `reproductor.jsx` | `GamePlayer` | `/games/[id]/play` — HUD + CRT frame; the game loop is a fake score ticker |
| `auth.jsx` | `Auth` | `/auth` — sign-in/sign-up, no backend |
| `salon.jsx` | `HallOfFame` | `/salon` — per-game leaderboards with podium |
| `styles.css` | — | all design tokens (`--cyan`, `--magenta`, `--pixel` font, CRT/scanline/grid effects) |

Prototype conventions worth preserving when porting:

- **UI copy is Spanish** (`lang="es"`, `toLocaleString("es-ES")`). Keep it Spanish; identifiers stay English/Spanish as in the templates.
- Routing is hash-based (`{ name, id }` JSON in `location.hash`) driven by a single `route` state in `app.jsx` — this is what App Router file routes replace.
- Session and scores are `localStorage` only: keys `av_user` (`{ name }`) and `av_scores`. There is no server, auth, or persistence layer yet.
- Game covers are pure CSS art (`cover-bricks`, `cover-snake`, …) keyed off `game.cover`; no image assets.
- Because the prototype has no build step, each file aliases hooks to avoid global collisions (`useState: useStateApp`, `useStateB`, …). That workaround is unnecessary in the ported code — drop it.

`references/resources/__MACOSX/` is macOS archive noise; ignore it.

## Workflow

Per `README.md`, this project follows spec-driven development using the `/spec` and `/spec-impl` skills from [Klerith/fernando-skills](https://github.com/Klerith/fernando-skills), installed with `npx skills@latest add Klerith/fernando-skills`. Those skills are not currently installed in this environment.
