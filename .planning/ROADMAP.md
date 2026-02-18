# Roadmap: PDF Analytics Dashboard

**Project start:** 2026-02-18
**Architecture:** Feature Sliced Design (FSD) — non-negotiable
**Workflow:** GSD (`/gsd:plan-phase N` → `/gsd:execute-phase N` → `/gsd:verify-work`)

---

## Milestone 1: FSD Foundation (v0.1)

**Goal:** A Feature-Sliced-Design-compliant React application scaffold with all 6 FSD layers populated, mocked data throughout, URL-based widget routing, a virtualized PDF viewer, and an auth shell — all verifiable by running `npm run dev`.

**Definition of done:**
- All 6 FSD layers exist with correct slice boundaries
- ESLint reports zero FSD import violations
- App boots, routes correctly, and all pages render with mocked data
- README.md and CLAUDE.md document the FSD architecture and GSD workflow

---

### Phase 1 — Project Bootstrap

**Goal:** A Vite + React 19 + TypeScript + Tailwind CSS v4 project starts locally, all v1 dependencies are installed, TypeScript path aliases for all 6 FSD layers are configured, ESLint enforces FSD import rules, and the FSD directory skeleton is created.

**Key deliverables:**
- `package.json` with all dependencies (see STACK.md)
- `vite.config.ts` with `@tailwindcss/vite`, `vite-tsconfig-paths`
- `tsconfig.json` with `strict: true` and FSD path aliases (`@app/*`, `@pages/*`, `@widgets/*`, `@features/*`, `@entities/*`, `@shared/*`)
- `src/app/styles/index.css` with `@import "tailwindcss"`
- `src/app/index.tsx` rendering "FSD Scaffold" placeholder
- `.eslintrc.json` with `@feature-sliced/eslint-config` rules
- FSD directory skeleton: `src/app/`, `src/pages/`, `src/widgets/`, `src/features/`, `src/entities/`, `src/shared/`
- `npm run dev` serves app at localhost

**Requirements covered:** SETUP-01..05, ARCH-01

**Dependencies:** None (first phase)

---

### Phase 2 — Shared Layer

**Goal:** The `shared/` layer is fully built — UI design-system primitives, a mock API client with simulated network delay, react-pdf worker setup, utility functions, route path constants, and environment config. No business logic lives in shared/.

**Key deliverables:**
- `shared/ui/`: Button, Input, Badge, Skeleton, Spinner, Toast, Modal, Card — Tailwind-styled, no business logic
- `shared/api/client.ts`: base HTTP client with mock interceptor (returns mock data after ~300ms delay)
- `shared/lib/pdf.ts`: `react-pdf` worker setup (`pdfjs.GlobalWorkerOptions.workerSrc`), exported for app bootstrap
- `shared/lib/cn.ts`: `clsx` + `tailwind-merge` utility
- `shared/config/routes.ts`: typed route path constants (`ROUTES.reports`, `ROUTES.report(id)`, `ROUTES.widgetTab(id, tab)`)
- `shared/config/env.ts`: typed `import.meta.env` access
- `shared/types/global.d.ts`: global module augmentations
- Every slice has `index.ts` public API

**Requirements covered:** ARCH-02, VIEW-05

**Dependencies:** Phase 1

---

### Phase 3 — Entities Layer

**Goal:** The three business-domain entities (report, user, widget-tab) are defined as TypeScript types, Zustand stores, mock data factories, and TanStack Query API functions. The cross-widget sync store `useReportViewStore` lives in `entities/report/model/` as the FSD-correct shared state pattern.

**Key deliverables:**
- `entities/report/`:
  - `model/report.types.ts`: `Report` interface, `ReportStatus` enum (`pending | ready | error`)
  - `model/report.store.ts`: Zustand store (`reports[]`, `selectedReport`, actions)
  - `model/report-view.store.ts`: `useReportViewStore` — cross-widget sync state (`activePage`, `highlightedWidgetId`, actions)
  - `api/reportKeys.ts`: TanStack Query key factory
  - `api/reportsApi.ts`: `reportListQuery()`, `reportDetailQuery()` — queries with mocked `queryFn`
  - `api/mockData.ts`: 3–5 mocked `Report` objects with realistic data
  - `ui/ReportCard.tsx`: card component (title, date, page count, status badge)
  - `index.ts`: public API
- `entities/user/`:
  - `model/user.types.ts`: `User`, `UserRole` types
  - `model/user.store.ts`: Zustand store (`currentUser`, `isAuthenticated`)
  - `index.ts`
- `entities/widget-tab/`:
  - `model/tab.types.ts`: `TabId = 'charts' | 'tables' | 'stats' | 'text'`
  - `model/tab.constants.ts`: `TABS` array with `{id, label, path}` entries
  - `index.ts`

**Requirements covered:** ARCH-03, ARCH-05

**Dependencies:** Phase 2

---

### Phase 4 — Features Layer

**Goal:** The four feature slices (auth-session, upload-pdf, filter-reports, zoom-pdf) are built as shell components with Zustand stores and mocked interactions. Each feature owns its user-facing UI and local state. All mocked — no real network calls.

**Key deliverables:**
- `features/auth-session/`:
  - `ui/LoginForm.tsx`: email + password fields + submit button (shell, no real validation)
  - `model/session.store.ts`: Zustand (`isAuthenticated`, `currentUser`, `login()`, `logout()`)
  - `api/auth.ts`: mocked `loginUser()` (resolves after delay, sets session)
  - `index.ts`
- `features/upload-pdf/`:
  - `ui/UploadZone.tsx`: drag-and-drop area + click-to-browse `<input>` (PDF-only validation)
  - `ui/UploadProgress.tsx`: animated progress bar (mocked 0→100%)
  - `model/upload.store.ts`: Zustand (`file`, `progress`, `status: idle|uploading|success|error`)
  - `api/uploadApi.ts`: mocked upload (resolves after fake progress)
  - `index.ts`
- `features/filter-reports/`:
  - `ui/ReportFilters.tsx`: search input + status filter dropdown
  - `model/filters.store.ts`: Zustand (`query`, `status`, `sort`)
  - `index.ts`
- `features/zoom-pdf/`:
  - `ui/ZoomControls.tsx`: zoom in / zoom out / fit-to-width buttons
  - `model/zoom.store.ts`: Zustand (`zoomLevel: number`, `setZoom()`, `fitToWidth()`)
  - `index.ts`

**Requirements covered:** AUTH-01, AUTH-03, AUTH-04, UPLD-01..06

**Dependencies:** Phase 3

---

### Phase 5 — Widgets Layer

**Goal:** The four widget slices (report-list, upload-modal, pdf-viewer, widget-panel) are self-contained UI blocks that compose lower-layer slices. The pdf-viewer uses virtualized rendering. The widget-panel renders 4 URL-based tab shells. All data is mocked.

**Key deliverables:**
- `widgets/report-list/`:
  - `ui/ReportList.tsx`: renders list of `ReportCard` entities; uses `filter-reports` feature; handles loading (Skeleton), empty state
  - `index.ts`
- `widgets/upload-modal/`:
  - `ui/UploadModal.tsx`: dialog shell containing `upload-pdf` feature's `UploadZone` + `UploadProgress`
  - `index.ts`
- `widgets/pdf-viewer/`:
  - `ui/PdfViewer.tsx`: `<Document>` from react-pdf + `useVirtualizer` from `@tanstack/react-virtual`; renders only visible pages; reads `activePage` from `useReportViewStore`
  - `ui/PdfPage.tsx`: renders a single page (`<Page>` from react-pdf + Skeleton placeholder)
  - `ui/PdfControls.tsx`: prev/next buttons, page number input, total page count; uses `zoom-pdf` feature's `ZoomControls`
  - `model/pdf-viewer.store.ts`: local Zustand state for scroll position, total pages
  - `index.ts`
- `widgets/widget-panel/`:
  - `ui/WidgetPanel.tsx`: tab nav bar + `<Outlet>` for active tab
  - `ui/tabs/ChartsTab.tsx`: Recharts `BarChart` + `LineChart` with mocked data
  - `ui/tabs/TablesTab.tsx`: sortable HTML table with mocked rows (click column header to sort)
  - `ui/tabs/StatsTab.tsx`: 3–4 KPI cards (big number + label + trend indicator)
  - `ui/tabs/TextTab.tsx`: mocked text excerpts in styled cards
  - `model/active-tab.store.ts`: syncs active tab with URL
  - `index.ts`

**Requirements covered:** RPTS-01..03, VIEW-01..04, WDGT-01..06, ROUT-04

**Dependencies:** Phase 4

---

### Phase 6 — Pages, App Layer & Documentation

**Goal:** The app layer bootstraps all providers and code-based TanStack Router routes. Pages assemble widgets into complete, navigable screen layouts. Auth guard protects all report routes. README.md and CLAUDE.md are updated to document the FSD architecture and GSD workflow.

**Key deliverables:**
- `app/providers.tsx`: `<QueryClientProvider>` + `<RouterProvider>` + pdf worker import
- `app/router.tsx`: code-based route tree — root route (with auth `beforeLoad` guard), `/login`, `/`, `/report/:reportId` (nested: `/charts`, `/tables`, `/stats`, `/text`), default redirect from `/report/:reportId` → `/charts`
- `app/index.tsx`: `ReactDOM.createRoot(...)` entry point
- `pages/login/ui/LoginPage.tsx`: centered login card using `features/auth-session` `LoginForm`
- `pages/reports/ui/ReportsPage.tsx`: header + `ReportList` widget + upload button trigger + `UploadModal` widget
- `pages/report/ui/ReportPage.tsx`: CSS grid split layout (`1fr 1fr`) — left: `WidgetPanel`, right: `PdfViewer`; reads `:reportId` from route params
- `pages/report/ui/ReportLayout.tsx`: shared wrapper for report page breadcrumbs
- `README.md`: updated (see DOCS-01)
- `CLAUDE.md`: updated (see DOCS-02)

**Requirements covered:** AUTH-02, RPTS-04, ROUT-01..05, ARCH-04, DOCS-01..02

**Dependencies:** Phase 5

---

## Milestone 2: Real Data & Backend (v0.2)

*Planned — not yet phased. Begins after Milestone 1 is shipped and architecture is validated.*

**Goal:** Replace all mocked data with a real backend API. Add PDF text extraction. Implement true widget-to-PDF sync with source page annotations.

**Themes:**
- Backend API (Node.js/FastAPI/similar) for report storage and retrieval
- PDF text + table extraction (pdfjs text layer or server-side pipeline)
- Widget data annotated with `sourcePageNumber` for real PDF sync
- Real file upload + storage (S3 or similar)
- Error boundaries, retry logic, optimistic UI
- Auth with real token validation (JWT or session cookie)

---
*Roadmap created: 2026-02-18*
*Last updated: 2026-02-18 after Milestone 1 scope definition*
