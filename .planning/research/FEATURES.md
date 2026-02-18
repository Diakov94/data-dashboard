# Feature Research

**Domain:** PDF Analytics Dashboard (interactive reports with visualizations)
**Researched:** 2026-02-18
**Confidence:** MEDIUM (features well-understood; FSD boundary judgments are reasoned conventions, not specs)

---

## Feature Landscape

This dashboard has two pages: (1) a Report Library page listing uploaded PDFs, and (2) a Report Detail page with a split layout — left panel (widget navigation + visualizations) and right panel (virtualized PDF viewer). v1 uses mocked data and FSD scaffold.

Features are grouped by functional area, then categorized as table stakes, differentiators, or anti-features within each group.

---

### Table Stakes (Users Expect These)

Features users assume exist. Missing these = product feels incomplete.

#### Report Library / List Management

| Feature | Why Expected | Complexity | FSD Layer | Notes |
|---------|--------------|------------|-----------|-------|
| Report list view (card or row) | Every file manager has a list | LOW | `widgets/report-list` | Show title, upload date, page count, status badge |
| Upload date / name sort | Standard in any list | LOW | `features/sort-reports` | Ascending/descending toggle per column |
| Status badge (processing / ready / error) | Upload states are non-obvious | LOW | `entities/report` | Enum: `pending`, `ready`, `error` on entity model |
| Search by report name | Expected in any list > 10 items | LOW | `features/search-reports` | Client-side filter on v1 mocked list |
| Filter by status | Users need to find broken uploads | LOW | `features/filter-reports` | Dropdown or chip filter; drives list slice |
| Empty state (no reports yet) | First-run UX must not be blank | LOW | `widgets/report-list` | CTA to upload first report |
| Loading state (skeleton or spinner) | API latency must be communicated | LOW | `shared/ui` | Skeleton preferred over spinner for list |
| Report count indicator | "Showing 12 reports" orientation | LOW | `widgets/report-list` | Plain text, updates with filters |

#### Upload Flow

| Feature | Why Expected | Complexity | FSD Layer | Notes |
|---------|--------------|------------|-----------|-------|
| Drag-and-drop drop zone | File upload UX standard since 2015 | LOW | `features/upload-pdf` | Dashed border, icon, label; visual feedback on hover |
| Click-to-browse fallback | Drag-and-drop inaccessible alone | LOW | `features/upload-pdf` | `<input type="file" accept=".pdf">` behind the drop zone |
| PDF-only validation (client-side) | Wrong file type must fail before upload | LOW | `features/upload-pdf` | Check MIME `application/pdf` AND `.pdf` extension |
| File size limit + error message | Prevent accidental huge uploads | LOW | `features/upload-pdf` | 50 MB default; show exact limit in error |
| Upload progress indicator | Without it users think app froze | LOW | `features/upload-pdf` | Progress bar with percentage; Google Drive pattern |
| Upload error state with retry | Uploads fail; user must recover | LOW | `features/upload-pdf` | Show error message + "Try again" button |
| Upload success feedback | Confirmation the file was received | LOW | `features/upload-pdf` | Toast or inline success state in list |

#### PDF Viewer (Right Panel)

| Feature | Why Expected | Complexity | FSD Layer | Notes |
|---------|--------------|------------|-----------|-------|
| Page rendering (all pages) | Core viewer function | MEDIUM | `widgets/pdf-viewer` | Use `react-pdf` (wojtekmaj); virtual rendering |
| Page navigation (prev/next, jump-to-page) | Non-linear navigation is standard | LOW | `widgets/pdf-viewer` | Input field + arrow buttons; keyboard shortcuts |
| Zoom in / out / reset | Documents must be readable | LOW | `widgets/pdf-viewer` | Fit-to-width and fit-to-page presets + custom % |
| Current page / total pages indicator | "Page 4 of 32" orientation | LOW | `widgets/pdf-viewer` | Simple text indicator near navigation |
| Virtualized rendering (visible pages only) | Required for large PDFs (100+ pages) | MEDIUM | `widgets/pdf-viewer` | Render ±2 pages around viewport; critical for perf |
| Loading state per page | Pages render asynchronously | LOW | `widgets/pdf-viewer` | Placeholder skeleton per page canvas |
| Text selection | Users copy numbers or excerpts | LOW | `widgets/pdf-viewer` | Enabled by default in `react-pdf` |
| Scroll position persistence | Navigating away and back | LOW | `widgets/pdf-viewer` | Store scroll offset in URL hash or local state |

#### Widget Navigation (Left Panel)

| Feature | Why Expected | Complexity | FSD Layer | Notes |
|---------|--------------|------------|-----------|-------|
| Sub-navigation tabs/links | Split layout implies multiple views | LOW | `widgets/report-nav` | Links to `/report/:id/charts`, `/tables`, `/stats`, `/text` |
| Active route highlight | User must know where they are | LOW | `widgets/report-nav` | React Router `NavLink` `isActive` class |
| Breadcrumbs | Deep URL means users get lost | LOW | `widgets/report-nav` | "Reports > Report Name > Charts" pattern |
| Widget panel scroll | Content may exceed viewport | LOW | `widgets/report-nav` | Left panel scrolls independently of PDF |

#### Data Visualizations (Chart / Table / KPI Widgets)

| Feature | Why Expected | Complexity | FSD Layer | Notes |
|---------|--------------|------------|-----------|-------|
| KPI summary cards (big number + trend) | First thing users look for | LOW | `widgets/kpi-cards` | Value, label, trend indicator (up/down %, color) |
| Bar chart for categorical comparison | Most common chart type | LOW | `widgets/charts` | Use Recharts; horizontal or vertical |
| Line chart for trends over time | Time-series data is universal | LOW | `widgets/charts` | With tooltip on hover |
| Data table with sortable columns | Tabular data must be navigable | MEDIUM | `widgets/data-table` | Column sort, fixed header, row highlight |
| Empty state per widget | Mocked data may have gaps | LOW | `widgets/charts` | "No data available" placeholder |
| Loading skeleton per widget | Async data fetch illusion | LOW | `shared/ui` | Skeleton card shape matching widget dimensions |

#### Authentication (Small Team)

| Feature | Why Expected | Complexity | FSD Layer | Notes |
|---------|--------------|------------|-----------|-------|
| Login page (email + password) | Any multi-user app requires this | LOW | `pages/login` | Simple form; no social login needed for v1 |
| Protected route guard | Unauthenticated access must be blocked | LOW | `features/auth-guard` | `<AuthGuard>` wrapper component; redirects to `/login` |
| Session persistence (remember me) | Users expect to stay logged in | LOW | `features/auth-session` | `httpOnly` cookie or `localStorage` JWT; pick one strategy |
| Logout action | Users must be able to end session | LOW | `features/auth-session` | Clear token + redirect to login |
| Auth state context | App-wide access to current user | LOW | `entities/viewer` | `AuthContext` or Zustand slice; `currentUser`, `isAuthenticated` |

---

### Differentiators (Competitive Advantage)

Features that set the product apart. Not required for v1, but valuable.

| Feature | Value Proposition | Complexity | FSD Layer | Notes |
|---------|-------------------|------------|-----------|-------|
| Widget-to-page sync | Clicking a chart highlights the source page in the PDF | HIGH | `features/widget-page-sync` | Requires data annotation mapping widget data → page number; most PDF viewers do not do this |
| Page thumbnails strip | Visual navigation of PDF pages | MEDIUM | `widgets/pdf-viewer` | Sidebar thumbnail strip; toggleable; useful for long docs |
| Minimap (document overview) | Shows position in long PDF | MEDIUM | `widgets/pdf-viewer` | Small viewport indicator on side; niche but polished |
| Annotation overlays on PDF | Highlight text that feeds a chart | HIGH | `widgets/pdf-viewer` | Requires coordinate-to-text mapping; v2 candidate |
| Saved filter presets | Users save their common list filters | MEDIUM | `features/filter-reports` | Store in `localStorage` or backend; reduces repeated work |
| Report tags / categories | Organize reports beyond status | MEDIUM | `entities/report` | User-defined tags; filter by tag |
| Column customization on data table | Power users want to show/hide columns | MEDIUM | `widgets/data-table` | Persisted column config per user |
| Keyboard shortcuts for PDF navigation | Power user productivity | LOW | `widgets/pdf-viewer` | Arrow keys, `+`/`-` zoom, `f` for fit |
| Dark mode | Reduces eye strain for long document sessions | MEDIUM | `app/theme` | CSS variables + theme context; user toggle |
| Export widget data (CSV) | Analysts want raw numbers | MEDIUM | `features/export-data` | Client-side CSV from mocked data; real export in v2 |
| Chart tooltips with rich context | Hover reveals source text or page | MEDIUM | `widgets/charts` | Recharts `custom tooltip`; links to PDF page |

---

### Anti-Features (Commonly Requested, Often Problematic)

Features that seem good but create problems — do not build these in v1.

| Feature | Why Requested | Why Problematic | Alternative |
|---------|---------------|-----------------|-------------|
| Real-time collaborative editing | Teams want to work together | Adds WebSocket infra, conflict resolution, operational transforms — months of work for v1 | Single-user sessions in v1; add collaboration in v2 with a proven library |
| In-browser PDF editing (redline, annotate-to-save) | Users want to mark up documents | PDF mutation is a completely different problem domain (pdf-lib, iText) from viewing; destroys scope | Read-only PDF viewing in v1; annotation as a discrete v2 milestone |
| Full-text PDF search (cross-document) | "Find this term in all my reports" | Requires PDF text extraction pipeline (pdfjs or Tika), indexing (Elasticsearch or Typesense), and search API — not frontend work | Per-page text selection + browser find (Ctrl+F) covers 80% of the use case |
| PDF signature / digital signing | Enterprise users ask for this | Requires PKI infrastructure, compliance (eIDAS, ESIGN), and a signing UX that is a product in itself | Out of scope; link to DocuSign or similar if needed |
| AI summary / chat with PDF | Trendy feature request | Requires LLM API budget, prompt engineering, latency management, and hallucination handling — none of which is dashboard-core work | Log the request; add in a dedicated AI milestone after core analytics is proven |
| Role-based permissions (admin/viewer/editor) | Requested once a second user exists | Over-engineered for small team v1; every role enforcement point becomes a bug surface | Simple authenticated/unauthenticated is sufficient; add roles only when team size demands it |
| Bulk report management (select all, bulk delete) | Appears on roadmaps early | Requires confirmation dialogs, undo state, and server batch endpoints — all distraction from core analytics | Single-report delete with confirmation is sufficient for v1 |
| Mobile-responsive PDF viewer | Marketing wants mobile | Virtualized PDF rendering + split-pane layout is fundamentally desktop-first; mobile PDF viewing is a separate interaction paradigm | Graceful degradation with a "best viewed on desktop" notice; mobile in v2 |

---

## Feature Dependencies

```
[Auth (login page + session)]
    └──required by──> [Route guard]
                           └──gates──> [Report Library page]
                                           └──hosts──> [Upload flow]
                                           └──hosts──> [Report list + search/filter/sort]
                                                           └──navigates to──> [Report Detail page]

[Report Detail page]
    └──hosts──> [Widget nav (tabs + breadcrumbs)]
    └──hosts──> [PDF Viewer widget]
    └──hosts──> [Widget views: charts | tables | stats | text]

[Widget views]
    └──requires──> [Recharts / data table setup in shared]
    └──optionally enhances via──> [Widget-page sync] (differentiator)

[Upload flow]
    └──requires──> [File validation (client-side)]
    └──requires──> [Progress indicator]
    └──produces──> [Report entity in list]

[PDF Viewer widget]
    └──requires──> [react-pdf setup + worker config]
    └──requires──> [Virtualized rendering for perf]
    └──enhances with──> [Page thumbnails] (differentiator)
    └──enhances with──> [Minimap] (differentiator)
    └──enhances with──> [Widget-page sync] (differentiator)
```

### Dependency Notes

- **Auth before routing:** Protected routes depend on auth state. Auth must be implemented first — or scaffolded with a mock user — before any page can be accessed in a realistic flow.
- **Report entity before UI:** The `entities/report` model (shape, status enum, mock data factory) must exist before list widgets and upload features can render anything meaningful.
- **react-pdf worker before viewer:** `react-pdf` requires a Web Worker setup (`GlobalWorkerOptions.workerSrc`). This must be done in `app/` initialization; forgetting it breaks all PDF rendering.
- **Recharts before charts widgets:** Chart widgets import from Recharts. Install and configure Recharts in `shared/` before building any chart widget.
- **URL structure before widget nav:** Route schema (`/report/:id/charts`, `/tables`, `/stats`, `/text`) must be decided before building `widgets/report-nav`, otherwise the links are wrong and require a rewrite.
- **Widget-page sync depends on data annotations:** This differentiator requires that each widget's data point carries a `sourcePageNumber` metadata field. If mocked data is built without this field, sync cannot be added later without reworking all mocks.

---

## FSD Layer Map (Full Reference)

This table maps every significant feature to its FSD home. Use this when creating slices.

| Slice | Layer | Responsibility |
|-------|-------|---------------|
| `report` | `entities` | Report data model, status enum, mock data factory, TypeScript type |
| `viewer` | `entities` | Current authenticated user model |
| `report-list` | `widgets` | Composed list: search + filter + sort + empty/loading states |
| `pdf-viewer` | `widgets` | PDF canvas render, navigation, zoom, virtualization, scroll persistence |
| `report-nav` | `widgets` | Left panel: sub-navigation tabs, breadcrumbs, panel scroll |
| `kpi-cards` | `widgets` | KPI card row: big number + trend indicator |
| `charts` | `widgets` | Bar chart, line chart with Recharts; tooltip; empty state |
| `data-table` | `widgets` | Sortable table with fixed header; row highlight |
| `upload-pdf` | `features` | Drag-drop zone, click-to-browse, validation, progress, error/success |
| `search-reports` | `features` | Search input driving list filter |
| `filter-reports` | `features` | Status + tag filter driving list |
| `sort-reports` | `features` | Column sort toggles |
| `auth-guard` | `features` | Route guard component; redirects unauthenticated users |
| `auth-session` | `features` | Login form submit, token storage, logout, session check |
| `widget-page-sync` | `features` | Maps widget data point → PDF page; triggers viewer scroll (differentiator, v1.x) |
| `export-data` | `features` | Client-side CSV export from widget data (differentiator, v1.x) |
| `ui` | `shared` | Button, Input, Badge, Skeleton, Toast, Modal — no business logic |
| `api` | `shared` | Axios/fetch client; base URL config; error interceptor |
| `config` | `shared` | Environment constants, route path constants |
| `lib/pdf` | `shared` | `react-pdf` worker setup, PDF utilities |
| `lib/charts` | `shared` | Recharts shared config, color palette, formatters |
| `login` | `pages` | Login page layout — composes `auth-session` feature |
| `reports` | `pages` | Report library page — composes `report-list` + `upload-pdf` |
| `report-detail` | `pages` | Split-layout page — composes `report-nav`, `pdf-viewer`, widget views |

---

## MVP Definition

### Launch With (v1)

Minimum viable product — validated concept with mocked data, full FSD scaffold.

- [ ] Auth: login page, session, logout, route guard — no real backend needed; mock `isAuthenticated = true` for dev
- [ ] Report Library page: list with status badges, search, filter by status, sort by name/date, empty state, loading state
- [ ] Upload flow: drag-drop zone, click-to-browse, PDF-only validation, file size check, progress bar, error/retry, success toast
- [ ] Report Detail page: split layout with URL routing (`/report/:id/charts` etc.), breadcrumbs, left-panel nav tabs
- [ ] KPI cards widget: 3–4 mocked KPI cards with trend indicators
- [ ] Bar chart widget: one bar chart with mocked data and Recharts
- [ ] Line chart widget: one line chart with mocked data
- [ ] Data table widget: sortable columns, mocked rows, fixed header
- [ ] PDF Viewer: renders all pages with virtualization, prev/next/jump-to-page navigation, zoom in/out/fit, text selection
- [ ] `shared/ui`: Button, Input, Badge, Skeleton, Toast components

### Add After Validation (v1.x)

Features to add once core is working and the split-layout pattern is proven.

- [ ] Widget-to-page sync — trigger when annotated mock data includes `sourcePageNumber`
- [ ] Page thumbnails strip — toggleable sidebar in PDF viewer
- [ ] Keyboard shortcuts for PDF navigation
- [ ] Export widget data as CSV
- [ ] Saved filter presets (localStorage)

### Future Consideration (v2+)

Features to defer until product-market fit is established.

- [ ] Real backend (replace mocked data with API)
- [ ] Full-text search across documents (requires extraction pipeline)
- [ ] Annotation overlays on PDF
- [ ] Report tags / categories
- [ ] Dark mode
- [ ] Mobile-responsive layout
- [ ] Role-based permissions
- [ ] AI summary / chat with PDF

---

## Feature Prioritization Matrix

| Feature | User Value | Implementation Cost | Priority |
|---------|------------|---------------------|----------|
| Report list + status badges | HIGH | LOW | P1 |
| Upload flow (drag-drop + validation + progress) | HIGH | LOW | P1 |
| PDF Viewer (render + navigation + zoom) | HIGH | MEDIUM | P1 |
| Split layout + URL routing + nav tabs | HIGH | MEDIUM | P1 |
| KPI cards widget | HIGH | LOW | P1 |
| Bar + line chart widgets | HIGH | LOW | P1 |
| Data table widget | HIGH | MEDIUM | P1 |
| Auth (login + guard + session) | HIGH | LOW | P1 |
| Breadcrumbs | MEDIUM | LOW | P1 |
| Search + filter + sort on report list | MEDIUM | LOW | P1 |
| Virtualized PDF rendering | HIGH | MEDIUM | P1 |
| Widget-page sync | HIGH | HIGH | P2 |
| Page thumbnails strip | MEDIUM | MEDIUM | P2 |
| Keyboard shortcuts | MEDIUM | LOW | P2 |
| Export CSV | MEDIUM | LOW | P2 |
| Saved filter presets | LOW | LOW | P2 |
| Dark mode | MEDIUM | MEDIUM | P3 |
| Annotation overlays | HIGH | HIGH | P3 |
| Full-text search | HIGH | HIGH | P3 |
| AI summary | MEDIUM | HIGH | P3 |
| Mobile layout | MEDIUM | HIGH | P3 |
| Role-based permissions | LOW | HIGH | P3 |

**Priority key:**
- P1: Must have for v1 launch
- P2: Should have, add in v1.x after core is stable
- P3: Future consideration, v2+

---

## Competitor Feature Analysis

Reference products: Datadog Dashboards, Retool, Google Looker Studio, Adobe Acrobat Pro (viewer), Power BI embedded.

| Feature | Datadog / Looker | Retool | Our Approach |
|---------|-----------------|--------|--------------|
| Report list | Tag-based, faceted search | Table with inline search | Simpler: status filter + name search; no tags in v1 |
| PDF viewing | None (data only) | File viewer exists | Core differentiator: PDF + analytics together |
| Chart types | Dozens (overkill) | Standard set | Bar + line + table + KPI cards = covers 95% of cases |
| Widget routing | Tab-based or sidebar | Page-based | URL-driven sub-navigation for shareability |
| Auth | SSO / SAML / LDAP | Auth providers | Simple email/password for small team |
| Upload | Drag-drop to data source | File component | PDF-specific with type/size validation |
| Widget-page sync | N/A | N/A | True differentiator — no competitor does PDF + chart sync |

---

## Sources

- Feature-Sliced Design official layers reference: https://feature-sliced.design/docs/reference/layers (MEDIUM confidence — verified via WebFetch)
- FSD overview and examples: https://feature-sliced.design/docs/get-started/overview (MEDIUM confidence)
- PDF viewer library comparison 2025: https://blog.react-pdf.dev/top-6-pdf-viewers-for-reactjs-developers-in-2025 (LOW confidence — single source)
- react-pdf GitHub: https://github.com/wojtekmaj/react-pdf (MEDIUM confidence)
- React PDF Viewer plugin docs (page navigation): https://react-pdf-viewer.dev/plugins/page-navigation/ (MEDIUM confidence)
- File upload UX best practices: https://uploadcare.com/blog/file-uploader-ux-best-practices/ (MEDIUM confidence — multiple sources corroborate)
- Drag-and-drop upload UX 2025: https://blog.filestack.com/building-modern-drag-and-drop-upload-ui/ (LOW confidence — single source)
- Dashboard UX best practices 2025: https://www.smashingmagazine.com/2025/09/ux-strategies-real-time-dashboards/ (MEDIUM confidence)
- KPI dashboard best practices: https://www.simplekpi.com/Blog/KPI-Dashboards-a-comprehensive-guide (MEDIUM confidence)
- Data visualization chart types 2025: https://www.owox.com/blog/articles/types-of-charts-and-graphs (LOW confidence)
- React auth small team 2025: https://workos.com/blog/top-authentication-solutions-react-2026 (MEDIUM confidence)
- Recharts 3.0 release 2025: https://recharts.org/ (MEDIUM confidence)
- Dashboard list management UX: https://vantagepoint.io/blog/va/data-control-advanced-filtering-sorting-search (LOW confidence)
- FSD widgets vs features boundary: https://www.codecentric.de/en/knowledge-hub/blog/feature-sliced-design-and-good-frontend-architecture (MEDIUM confidence)

---
*Feature research for: PDF Analytics Dashboard*
*Researched: 2026-02-18*
