# PDF Analytics Dashboard

## What This Is

An interactive analytics dashboard for PDF reports built for small teams. Users upload PDF documents that become navigable reports — each report opens in a split-panel view with interactive data widgets on the left and a synchronized PDF viewer on the right. The application is architected using Feature Sliced Design (FSD) as a non-negotiable structural constraint.

## Core Value

A user can upload a PDF and immediately navigate its data through widgets — charts, tables, stats, annotations — all synchronized with a virtualized PDF viewer that never lags.

## Requirements

### Validated

- ✓ Node.js project scaffold with package.json — existing

### Active

- [ ] FSD-compliant directory structure (app / pages / widgets / features / entities / shared layers)
- [ ] React + Vite application shell with TypeScript configured
- [ ] TanStack Router with nested routes (main, report/:id, report/:id/:widget, auth)
- [ ] TanStack Query for async data management with mocked API
- [ ] Zustand stores per FSD slice (entity and feature level)
- [ ] Tailwind CSS configured with design tokens in shared layer
- [ ] Authentication shell — login page, route guards, mocked credentials
- [ ] Report list page — displays mocked reports with upload trigger
- [ ] Upload report feature — file picker UI, progress indicator (mocked, no real upload)
- [ ] Report detail page — split layout (left widgets panel, right PDF viewer)
- [ ] Widget panel with URL-based sublinks (/report/:id/charts, /tables, /stats, /text)
- [ ] Chart widget shell (mocked chart data)
- [ ] Table widget shell (mocked table data)
- [ ] Summary stats widget shell (mocked KPIs)
- [ ] Text/annotations widget shell (mocked extracted text)
- [ ] PDF viewer widget — virtualized page list (renders only visible pages)
- [ ] PDF page minimap — right-side thumbnail strip for page jumping
- [ ] Zoom controls — zoom in/out and fit-to-width
- [ ] Widget-to-PDF sync — clicking a widget scrolls/highlights the related PDF page
- [ ] README.md updated with FSD architecture guide and GSD workflow notes
- [ ] CLAUDE.md updated with FSD constraints and GSD flow guidance

### Out of Scope

- Real PDF parsing / data extraction — deferred; v1 uses mocked data throughout
- Backend / API server — deferred; all data is client-side mocks
- Real file storage — deferred; upload UI exists but files are not persisted
- Real-time collaboration — not planned for v1
- Mobile / responsive layout — web-first, desktop viewport only for v1
- OAuth / SSO — simple mocked auth sufficient for v1
- PDF annotation editing — view-only in v1

## Context

The project starts as a bare Node.js scaffold (`index.js` placeholder, no source code). The entire application will be built from scratch using the FSD architecture pattern.

**Architecture mandate:** Feature Sliced Design (FSD) — https://feature-sliced.design/docs/get-started/overview — is the non-negotiable architectural pattern. Every file must live in the correct FSD layer. Cross-layer import rules must be enforced.

**FSD layer map for this project:**
- `src/app/` — Providers (QueryClient, Router), global styles, app bootstrap
- `src/pages/` — auth (login), main (report list), report (detail)
- `src/widgets/` — pdf-viewer, widget-panel, report-card, chart-widget, table-widget, stat-widget, text-widget
- `src/features/` — upload-report, navigate-widgets, sync-pdf-page, auth, zoom-pdf
- `src/entities/` — report, user, widget-data
- `src/shared/` — ui (primitive components), api (mock client), lib (pdf.js wrapper, react-virtual), types, config

**Cross-slice communication pattern:** Widget-PDF sync uses a shared Zustand store in `shared/model/` that both the `pdf-viewer` widget and `widget-panel` widget connect to, respecting FSD import rules.

**Coding principles (non-negotiable):**
- SOLID, DRY, KISS
- No code smells or antipatterns
- Follow React/Meta best practices (composition over inheritance, colocation, etc.)

## Constraints

- **Architecture**: Feature Sliced Design — every file must live in correct FSD layer, no cross-layer rule violations
- **Tech Stack**: React + TypeScript + Vite + Zustand + TanStack Router + TanStack Query + Tailwind CSS — no deviations without explicit discussion
- **Scope**: v1 is structure + mocked shell, not production-ready features
- **Import rules**: FSD layers may only import from layers below them (app→pages→widgets→features→entities→shared)

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Feature Sliced Design as architecture | Scalability, clear boundaries, team-friendly | — Pending |
| Zustand per-FSD-slice stores | Aligns with FSD slice ownership, no global god store | — Pending |
| TanStack Router with nested routes | URL-based widget sublinks require proper nested routing | — Pending |
| v1 = scaffold with mocked data | Ship structure fast, validate architecture before real backend | — Pending |
| Widget-PDF sync via shared store | Only FSD-compliant way to communicate across sibling widgets | — Pending |
| react-virtual for PDF virtualization | Prevents lag on large PDFs without full rendering | — Pending |

---
*Last updated: 2026-02-18 after initialization*
