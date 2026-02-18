# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## Project

Interactive analytics dashboard for PDF reports built for small teams. Users upload PDF documents that become navigable reports — each report opens in a split-panel view with interactive data widgets on the left and a synchronized virtualized PDF viewer on the right.

**Entry point:** `src/app/index.tsx` (React app via Vite)
**Package main:** `index.js` (placeholder only — do not use for app logic)

---

## Architecture: Feature Sliced Design (FSD)

**FSD is the mandatory, non-negotiable architecture for this project.**
All code must live in the correct FSD layer and slice. No exceptions.

Reference: https://feature-sliced.design/docs/get-started/overview

### Layers (top to bottom, import direction: top imports from below)

```
src/app/       — Bootstrapping: router, providers, global CSS, pdf.js worker init
src/pages/     — One slice per route; thin assemblers only — no business logic
src/widgets/   — Self-contained UI blocks; no page or other-widget imports
src/features/  — User-facing interactions (actions, mutations, local feature state)
src/entities/  — Business domain objects (models, stores, mock APIs, shared stores)
src/shared/    — Zero business logic: UI primitives, HTTP client, lib wrappers, config
```

### Import Rules (ESLint enforces these — violations are errors)

- Each layer may only import from layers **strictly below** it
- Same-layer imports between slices are **forbidden** (except entities via `@x` notation)
- `shared/` imports nothing from the project
- Every slice must have `index.ts` as its single public API — never import internal paths

```typescript
// ✅ Correct
import { useReportStore } from '@entities/report'

// ❌ Wrong — internal import bypasses public API
import { useReportStore } from '@entities/report/model/report.store'
```

### Cross-Widget Communication Pattern

Two widgets cannot import each other. Use a shared Zustand store in `entities/` instead:

```
widgets/widget-panel → entities/report → useReportViewStore.setActivePage()
widgets/pdf-viewer   → entities/report → useReportViewStore.activePage (reads + scrolls)
```

### Zustand Store Placement

- Each FSD slice with client state owns **one Zustand store** in its `model/` segment
- Stores are exported only via `index.ts` public API
- No global "app store" — cross-slice state is lifted to the lowest common FSD ancestor layer
- Server state (async data) → TanStack Query; client UI state → Zustand

### FSD Slice Structure (per slice)

```
slice-name/
├── ui/          — React components for this slice
├── model/       — Zustand store(s), types, constants
├── api/         — TanStack Query functions, query keys, mock data
└── index.ts     — Public API (the ONLY file other slices import from)
```

---

## Workflow: GSD (Get Shit Done)

This project uses the **GSD workflow** to manage planning and execution. All planning documents live in `.planning/`.

### GSD Phase Flow

```
/gsd:plan-phase N    → Research + plan Phase N (creates .planning/phaseN/PLAN.md)
/gsd:execute-phase N → Execute all tasks in the phase plan (atomic commits)
/gsd:verify-work     → Verify phase against its goal (creates VERIFICATION.md)
/gsd:progress        → Show current state and route to next action
```

### Milestone & Phase Structure

**Milestone 1: FSD Foundation (v0.1)** — in progress

| Phase | Name | Status |
|-------|------|--------|
| 1 | Project Bootstrap | Pending |
| 2 | Shared Layer | Pending |
| 3 | Entities Layer | Pending |
| 4 | Features Layer | Pending |
| 5 | Widgets Layer | Pending |
| 6 | Pages, App Layer & Documentation | Pending |

### Starting Work

```bash
# Plan Phase 1 (do this first)
/gsd:plan-phase 1

# Execute a planned phase
/gsd:execute-phase 1

# Check project status
/gsd:progress
```

---

## Commands

| Command | What It Does |
|---------|-------------|
| `npm run dev` | Start Vite dev server (available after Phase 1) |
| `npm run build` | Production build (available after Phase 1) |
| `npx tsc --noEmit` | Type-check all TypeScript files |
| `npx steiger src` | Run FSD architectural linter |
| `npx eslint src` | Run ESLint (includes FSD import rule enforcement) |

> **Before Phase 1 is complete:** `npm run dev` serves the bare Node.js `index.js` placeholder, not the React app.

---

## Tech Stack

| Technology | Version | Purpose |
|-----------|---------|---------|
| React | 19.x | UI framework |
| TypeScript | 5.x (`strict: true`) | Type safety; required for TanStack Router full type inference |
| Vite | 7.x | Build tool + dev server |
| TanStack Router | 1.x (code-based) | Client routing; code-based to avoid file-based conflict with FSD pages/ |
| TanStack Query | 5.x | Server state; queries live in entities/*/api and features/*/api |
| Zustand | 5.x | Client state; one store per FSD slice in model/ segment |
| Tailwind CSS | v4 | Styling; `@tailwindcss/vite` plugin (NOT PostCSS) |
| react-pdf (wojtekmaj) | 10.x | PDF rendering; wraps pdfjs-dist 5.x |
| @tanstack/react-virtual | 3.x | Virtualizes PDF page list; only visible pages rendered |
| Recharts | 2.x | Charts for widget panel |
| Steiger | latest | FSD architectural linter (official) |
| @feature-sliced/eslint-config | latest | ESLint FSD import rules |
| vite-tsconfig-paths | 5.x | Syncs tsconfig paths into Vite; define aliases once |

---

## MCP Servers

Two MCP servers are configured in `.mcp.json` (project-level, committed):

- **context7** — Library documentation lookup. Requires `CONTEXT7_API_KEY` env var (`export CONTEXT7_API_KEY=...`). Use when researching library APIs before implementing.
- **sequential-thinking** — Structured reasoning tool. No credentials required. Use for complex architectural decisions and phase planning.

---

## Key Constraints

1. **FSD is mandatory** — no code outside the layer/slice model
2. **No cross-layer violations** — ESLint will flag these as errors; fix before committing
3. **No same-layer widget imports** — widgets cannot import other widgets
4. **Public API only** — always import from `@layer/slice`, never `@layer/slice/internal/path`
5. **Zustand per slice** — one store per slice; no god store
6. **Code-based routing** — never use TanStack Router's file-based mode (breaks FSD pages/ convention)
7. **Mock data in v1** — no real backend; all data comes from mock factories in entities/*/api
8. **react-pdf worker setup once** — configure in `shared/lib/pdf.ts`, import in `app/providers.tsx`; never inside a component

---

## Planning Documents

| Document | Path | Purpose |
|----------|------|---------|
| Project context | `.planning/PROJECT.md` | What, why, for whom; core value; key decisions |
| Requirements | `.planning/REQUIREMENTS.md` | All v1 requirements with IDs (SETUP, ARCH, AUTH, RPTS, UPLD, VIEW, WDGT, ROUT, DOCS) |
| Roadmap | `.planning/ROADMAP.md` | Milestone 1 phases with goals and deliverables |
| State | `.planning/STATE.md` | Current phase, history, open questions |
| Architecture research | `.planning/research/ARCHITECTURE.md` | FSD layer map, patterns, anti-patterns, data flow |
| Stack research | `.planning/research/STACK.md` | Technology choices with versions and FSD integration patterns |
| Feature research | `.planning/research/FEATURES.md` | Feature landscape, FSD layer assignments, MVP definition |

---

## Anti-Patterns — Never Do These

| Anti-Pattern | Why | Correct Approach |
|---|---|---|
| `widgets/pdf-viewer` imports `widgets/widget-panel` | Same-layer violation | Both read from `entities/report` Zustand store |
| Business logic in `shared/` | shared/ must be project-agnostic | Move domain logic to entities or features |
| Pages containing query calls or large UI | Pages become monoliths | Pages are thin assemblers; keep page components <50 lines |
| Skipping `index.ts` public API | Breaks encapsulation; tooling can't enforce | Every slice needs `index.ts` before any external import |
| Global Zustand store | Defeats FSD slice isolation | Per-slice stores; lift shared state to lower FSD layer |
| File-based TanStack Router | `src/routes/` conflicts with FSD `src/pages/` | Code-based routing in `app/router.tsx` |
| pdf.js worker setup in a component | Non-deterministic module execution with code-splitting | Configure in `shared/lib/pdf.ts`, import in `app/providers.tsx` |
