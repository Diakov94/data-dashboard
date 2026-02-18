# Requirements: PDF Analytics Dashboard

**Defined:** 2026-02-18
**Core Value:** A user can upload a PDF and immediately navigate its data through widgets — charts, tables, stats, annotations — all synchronized with a virtualized PDF viewer that never lags.

---

## v1 Requirements

Milestone 1 scope: FSD-compliant scaffold with mocked data. No real backend. Proves architecture and UX structure.

### Setup & Configuration

- [ ] **SETUP-01**: Developer runs `npm install && npm run dev` and sees a React app in the browser
- [ ] **SETUP-02**: TypeScript compiles without errors (`tsc --noEmit`) across all FSD layers
- [ ] **SETUP-03**: FSD path aliases resolve correctly (`@app/*`, `@pages/*`, `@widgets/*`, `@features/*`, `@entities/*`, `@shared/*`)
- [ ] **SETUP-04**: Tailwind CSS v4 utility classes apply correctly in React components
- [ ] **SETUP-05**: ESLint reports FSD layer import violations as errors (no cross-layer rule violations in codebase)

### Architecture (FSD Compliance)

- [ ] **ARCH-01**: FSD directory skeleton exists for all 6 layers — `src/app/`, `src/pages/`, `src/widgets/`, `src/features/`, `src/entities/`, `src/shared/`
- [ ] **ARCH-02**: Every slice in every layer has an `index.ts` public API file (no direct internal path imports from outside a slice)
- [ ] **ARCH-03**: Zustand stores live in the `model/` segment of each slice that needs client state
- [ ] **ARCH-04**: Code-based TanStack Router (not file-based) to avoid conflict with FSD `pages/` layer convention
- [ ] **ARCH-05**: Cross-widget state coordination uses shared Zustand store in `entities/report/model/` (not shared/, not a global god store)

### Authentication

- [ ] **AUTH-01**: User sees a login page at `/login` with email + password form fields and a submit button
- [ ] **AUTH-02**: Unauthenticated user navigating to any protected route is redirected to `/login`
- [ ] **AUTH-03**: Submitting mocked valid credentials (any non-empty email/password) marks the session as authenticated
- [ ] **AUTH-04**: User can log out — session clears and user is redirected to `/login`

### Report Library

- [ ] **RPTS-01**: User sees a list of mocked reports on the main page (`/`) — each card shows title, upload date, page count, status badge
- [ ] **RPTS-02**: Report list shows an empty state with a CTA to upload when no reports exist
- [ ] **RPTS-03**: Report list shows a loading skeleton while data is "fetching" (simulated delay in mock)
- [ ] **RPTS-04**: Clicking a report card navigates to the report detail page (`/report/:reportId`)

### Upload Flow

- [ ] **UPLD-01**: A button on the reports list page opens an upload modal/dialog
- [ ] **UPLD-02**: Modal has a drag-and-drop drop zone with visual hover feedback
- [ ] **UPLD-03**: Modal has a click-to-browse fallback (`<input type="file" accept=".pdf">`)
- [ ] **UPLD-04**: Non-PDF files are rejected client-side with an error message (checks MIME type and `.pdf` extension)
- [ ] **UPLD-05**: A mocked upload progress bar appears and fills to 100% after file selection
- [ ] **UPLD-06**: Upload success state shows a toast notification; error state shows a retry button

### PDF Viewer

- [ ] **VIEW-01**: Report detail page right panel renders PDF pages using `react-pdf`
- [ ] **VIEW-02**: PDF page list is virtualized with `@tanstack/react-virtual` — only visible pages are rendered (critical for large PDFs)
- [ ] **VIEW-03**: Zoom in / zoom out / fit-to-width controls change the rendered PDF scale
- [ ] **VIEW-04**: Prev / Next page buttons and a jump-to-page number input navigate through pages
- [ ] **VIEW-05**: `react-pdf` Web Worker is configured at app bootstrap in `shared/lib/pdf` and imported once in `app/providers.tsx`

### Widget Panel

- [ ] **WDGT-01**: Report detail page left panel has 4 URL-based navigation tabs: Charts, Tables, Stats, Text
- [ ] **WDGT-02**: Navigating to `/report/:id/charts` renders a Charts tab with mocked bar chart + line chart (Recharts)
- [ ] **WDGT-03**: Navigating to `/report/:id/tables` renders a Tables tab with a mocked sortable data table
- [ ] **WDGT-04**: Navigating to `/report/:id/stats` renders a Stats tab with mocked KPI cards (big number + trend indicator)
- [ ] **WDGT-05**: Navigating to `/report/:id/text` renders a Text tab with mocked extracted text / annotation excerpts
- [ ] **WDGT-06**: Clicking a widget data point (e.g., chart annotation with `sourcePageNumber`) scrolls the PDF viewer to the referenced page via the shared `useReportViewStore`

### Routing

- [ ] **ROUT-01**: TanStack Router code-based route tree defined in `app/router.tsx`
- [ ] **ROUT-02**: Routes defined: `/login`, `/` (report list), `/report/:reportId` (detail), nested: `/report/:reportId/charts|tables|stats|text`
- [ ] **ROUT-03**: Default redirect from `/report/:reportId` to `/report/:reportId/charts`
- [ ] **ROUT-04**: Widget tab links use TanStack Router `<Link>` — tabs are deep-linkable and browser back/forward works
- [ ] **ROUT-05**: Auth guard implemented as TanStack Router `beforeLoad` hook on the root route

### Documentation

- [ ] **DOCS-01**: `README.md` describes the project, FSD architecture (layer map, import rules, slice directory), tech stack, setup commands, and GSD workflow reference
- [ ] **DOCS-02**: `CLAUDE.md` documents: FSD as the mandatory architecture, FSD import rules enforced by ESLint, GSD workflow (use `/gsd:plan-phase` to plan, `/gsd:execute-phase` to build, etc.), available npm commands, and MCP server usage

---

## v2 Requirements

Deferred. Tracked but not in current roadmap.

### Real Backend

- **BACK-01**: Replace mock API client with real REST/GraphQL API calls
- **BACK-02**: PDF file upload persisted to object storage
- **BACK-03**: Report metadata stored in a database

### PDF Analysis

- **ANLY-01**: Real PDF text extraction (pdfjs text layer or server-side Tika)
- **ANLY-02**: Real chart data extracted from PDF tables
- **ANLY-03**: Widget data annotated with source page numbers (enables real widget-PDF sync)

### Enhanced Features

- **FEAT-01**: Widget-to-PDF page highlight overlay (annotation on source text)
- **FEAT-02**: Page thumbnail strip in PDF viewer (toggleable sidebar)
- **FEAT-03**: Keyboard shortcuts for PDF navigation (arrow keys, +/- zoom, f for fit)
- **FEAT-04**: Export widget data as CSV
- **FEAT-05**: Saved filter presets (localStorage)
- **FEAT-06**: Dark mode (CSS variables + theme toggle)
- **FEAT-07**: Report tags / categories
- **FEAT-08**: Full-text search across documents

---

## Out of Scope

Explicitly excluded. Documented to prevent scope creep.

| Feature | Reason |
|---------|--------|
| Real PDF parsing / data extraction | v1 uses mocked data; backend required for real extraction |
| Backend / API server | All data is client-side mocks; no server needed for v1 |
| Real file storage | Upload UI exists but files are not persisted in v1 |
| Real-time collaboration | High complexity; not core value for v1 |
| Mobile / responsive layout | Desktop-first; split-panel layout is desktop-native |
| OAuth / SSO | Simple mocked email+password sufficient for small team v1 |
| PDF annotation editing | View-only in v1; editing is a different product scope |
| AI summary / chat with PDF | Requires LLM infrastructure; out of scope for analytics dashboard |
| Role-based permissions | Simple authenticated/unauthenticated sufficient for v1 |
| Bulk report management | Single-report delete with confirmation is sufficient for v1 |

---

## Traceability

| Requirement | Phase | Status |
|-------------|-------|--------|
| SETUP-01 | Phase 1 | Pending |
| SETUP-02 | Phase 1 | Pending |
| SETUP-03 | Phase 1 | Pending |
| SETUP-04 | Phase 1 | Pending |
| SETUP-05 | Phase 1 | Pending |
| ARCH-01 | Phase 1 | Pending |
| ARCH-02 | Phase 2 | Pending |
| ARCH-03 | Phase 3 | Pending |
| ARCH-04 | Phase 6 | Pending |
| ARCH-05 | Phase 3 | Pending |
| AUTH-01 | Phase 4 | Pending |
| AUTH-02 | Phase 6 | Pending |
| AUTH-03 | Phase 4 | Pending |
| AUTH-04 | Phase 4 | Pending |
| RPTS-01 | Phase 5 | Pending |
| RPTS-02 | Phase 5 | Pending |
| RPTS-03 | Phase 5 | Pending |
| RPTS-04 | Phase 6 | Pending |
| UPLD-01 | Phase 4 | Pending |
| UPLD-02 | Phase 4 | Pending |
| UPLD-03 | Phase 4 | Pending |
| UPLD-04 | Phase 4 | Pending |
| UPLD-05 | Phase 4 | Pending |
| UPLD-06 | Phase 4 | Pending |
| VIEW-01 | Phase 5 | Pending |
| VIEW-02 | Phase 5 | Pending |
| VIEW-03 | Phase 5 | Pending |
| VIEW-04 | Phase 5 | Pending |
| VIEW-05 | Phase 2 | Pending |
| WDGT-01 | Phase 5 | Pending |
| WDGT-02 | Phase 5 | Pending |
| WDGT-03 | Phase 5 | Pending |
| WDGT-04 | Phase 5 | Pending |
| WDGT-05 | Phase 5 | Pending |
| WDGT-06 | Phase 5 | Pending |
| ROUT-01 | Phase 6 | Pending |
| ROUT-02 | Phase 6 | Pending |
| ROUT-03 | Phase 6 | Pending |
| ROUT-04 | Phase 5 | Pending |
| ROUT-05 | Phase 6 | Pending |
| DOCS-01 | Phase 6 | Pending |
| DOCS-02 | Phase 6 | Pending |

**Coverage:**
- v1 requirements: 42 total
- Mapped to phases: 42
- Unmapped: 0 ✓

---
*Requirements defined: 2026-02-18*
*Last updated: 2026-02-18 after initial definition*
