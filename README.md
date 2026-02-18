# PDF Analytics Dashboard

Interactive analytics dashboard for PDF reports. Upload a PDF, generate a report, and explore its data through synchronized widgets and a virtualized PDF viewer.

---

## Architecture: Feature Sliced Design (FSD)

This project uses **[Feature Sliced Design (FSD)](https://feature-sliced.design/docs/get-started/overview)** as its architecture. FSD is a non-negotiable constraint — all code must live in the correct layer and slice.

### Layer Map

```
src/
├── app/          — Bootstrapping only: router, providers, global styles, pdf.js worker init
├── pages/        — One slice per route. Thin assemblers — pages render widgets, nothing else.
├── widgets/      — Large self-contained UI blocks: pdf-viewer, widget-panel, report-list, upload-modal
├── features/     — User-facing interactions: auth-session, upload-pdf, filter-reports, zoom-pdf
├── entities/     — Business domain objects: report (model + store + API), user, widget-tab
└── shared/       — Zero business logic: ui primitives, mock API client, pdf lib wrapper, config, types
```

### Import Rules (Enforced by ESLint)

Each layer may only import from layers **below** it. Same-layer imports are forbidden between slices.

```
app       → pages, widgets, features, entities, shared
pages     → widgets, features, entities, shared
widgets   → features, entities, shared
features  → entities, shared
entities  → shared  (+ other entities via @x public API only)
shared    → nothing
```

Violations are reported as ESLint errors via `@feature-sliced/eslint-config`.

### Public API Rule

Every slice exposes exactly one public `index.ts` file. External code imports **only** from this file.

```typescript
// ✅ Correct
import { useReportStore } from '@entities/report'

// ❌ Wrong — bypasses public API
import { useReportStore } from '@entities/report/model/report.store'
```

### Slice Map

| Slice | Layer | Responsibility |
|-------|-------|---------------|
| `app` | app | Bootstrap: router, providers, pdf worker, global CSS |
| `reports` | pages | Report list page — assembles report-list + upload-modal |
| `report` | pages | Report detail page — split layout: widget-panel (left) + pdf-viewer (right) |
| `login` | pages | Login page — assembles auth-session LoginForm |
| `pdf-viewer` | widgets | Right panel: virtualized PDF render, zoom, page nav, minimap |
| `widget-panel` | widgets | Left panel: URL-based tab nav (charts/tables/stats/text) + Outlet |
| `report-list` | widgets | Report cards list with filters, empty state, loading skeleton |
| `upload-modal` | widgets | PDF upload dialog — drag-drop + progress |
| `auth-session` | features | Login/logout, session Zustand store, mocked auth API |
| `upload-pdf` | features | Drag-drop zone, file validation, mocked upload progress |
| `filter-reports` | features | Search input + status filter + sort — drives report list |
| `zoom-pdf` | features | Zoom controls — in/out/fit-to-width |
| `report` | entities | Report type, status enum, mock data, Zustand store, useReportViewStore |
| `user` | entities | User type, session store |
| `widget-tab` | entities | TabId type, TABS constant (charts/tables/stats/text + paths) |
| `ui` | shared | Design system: Button, Input, Badge, Skeleton, Toast, Modal, Card |
| `api` | shared | Mock HTTP client with simulated delay |
| `lib/pdf` | shared | react-pdf worker setup (initialized once at app bootstrap) |
| `lib/cn` | shared | clsx + tailwind-merge utility |
| `config` | shared | Route path constants, environment variable access |

### Cross-Widget Communication

Two sibling widgets cannot import each other (same-layer rule). To sync the PDF viewer with the widget panel, both widgets read/write a shared Zustand store in `entities/report/model/report-view.store.ts` — a domain concept that belongs in the entity layer.

```
widgets/widget-panel  →  entities/report (useReportViewStore.setActivePage)
                                ↑
widgets/pdf-viewer    →  entities/report (useReportViewStore.activePage)
```

---

## Tech Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| UI | React 19 + TypeScript | Component framework |
| Build | Vite 7 | Dev server + bundler |
| Routing | TanStack Router (code-based) | Type-safe routing; avoids file-based conflict with FSD pages/ |
| Server state | TanStack Query v5 | Fetch, cache, sync async data |
| Client state | Zustand v5 (per-slice stores) | Local UI state per FSD slice |
| Styling | Tailwind CSS v4 | Utility-first CSS; `@tailwindcss/vite` plugin |
| PDF display | react-pdf (wojtekmaj) | Wraps pdf.js with React components |
| Virtualization | @tanstack/react-virtual | Renders only visible PDF pages (prevents lag on 100+ page PDFs) |
| Charts | Recharts | Bar, line charts for widget analytics |
| FSD linting | Steiger + @feature-sliced/eslint-config | Enforce FSD import rules at editor + CI level |
| Path aliases | vite-tsconfig-paths | Reads `tsconfig.json` paths; no duplicate config |

---

## Getting Started

```bash
# Install dependencies
npm install

# Start dev server
npm run dev

# Type-check
npx tsc --noEmit

# Run FSD linter
npx steiger src

# Build for production
npm run build
```

> **Note:** The project is in its scaffold stage. `npm run dev` currently serves a placeholder page. Run `/gsd:plan-phase 1` to start building.

---

## Project Management: GSD Workflow

This project is managed using the **GSD (Get Shit Done)** workflow for Claude Code. All planning documents live in `.planning/`.

### Planning Documents

| Document | Purpose |
|----------|---------|
| `.planning/PROJECT.md` | Living project context — what, why, for whom |
| `.planning/REQUIREMENTS.md` | All v1 requirements with IDs and traceability |
| `.planning/ROADMAP.md` | Milestone and phase structure |
| `.planning/STATE.md` | Current phase, history, open questions |
| `.planning/research/` | Architecture, stack, features, pitfalls research |

### GSD Commands

```bash
# Plan the next phase (creates PLAN.md with task breakdown)
/gsd:plan-phase 1

# Execute all tasks in a planned phase
/gsd:execute-phase 1

# Verify a completed phase against its goal
/gsd:verify-work

# Check project progress and get routing
/gsd:progress

# Add a todo from conversation context
/gsd:add-todo
```

### Phase Structure (Milestone 1)

| Phase | Name | Status |
|-------|------|--------|
| 1 | Project Bootstrap | Pending |
| 2 | Shared Layer | Pending |
| 3 | Entities Layer | Pending |
| 4 | Features Layer | Pending |
| 5 | Widgets Layer | Pending |
| 6 | Pages, App Layer & Documentation | Pending |

---

## Coding Principles

All code must follow:
- **SOLID** — single responsibility, open/closed, Liskov substitution, interface segregation, dependency inversion
- **DRY** — don't repeat yourself; abstract shared logic into lower FSD layers
- **KISS** — keep it simple; avoid premature abstraction
- **React best practices** — composition over inheritance, colocate state with its consumers, avoid prop drilling (use FSD slice stores instead), avoid unnecessary re-renders

Anti-patterns actively avoided: God stores, skipping index.ts public APIs, business logic in shared/, widgets importing each other, pages accumulating business logic.

---

## Contributing

1. All new code must live in the correct FSD layer — check the layer map above
2. Run `npx steiger src` before committing to catch FSD violations
3. Follow the GSD workflow — plan a phase before executing it
4. Imports must use `@layer/slice` path aliases, never relative paths across layers
5. Every new slice needs an `index.ts` public API before any code imports from it
