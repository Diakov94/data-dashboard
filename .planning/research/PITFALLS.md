# Pitfalls Research

**Domain:** React PDF Analytics Dashboard with Feature Sliced Design (FSD)
**Researched:** 2026-02-18
**Confidence:** MEDIUM-HIGH (FSD official docs + GitHub issues verified; some findings WebSearch-corroborated)

---

## Critical Pitfalls

### Pitfall 1: FSD Layer Violations at the `shared` Layer — The "God Shared" Anti-Pattern

**What goes wrong:**
Developers treating `shared/` as a dumping ground for anything that "seems reusable." Over time `shared/lib`, `shared/api`, and `shared/ui` accumulate unrelated logic — feature-specific constants, business domain types, API helpers that belong in `entities/`, and UI components tightly coupled to specific widgets. Because `shared` is the lowest layer with no restriction on who can import it, violations here silently propagate upward into every layer.

**Why it happens:**
FSD's most common escape hatch is "just put it in shared." When two features need the same thing and developers don't know which layer owns it, shared wins by default. This is correct for truly cross-cutting concerns (formatting utilities, base UI primitives, HTTP client config), but wrong for domain logic.

**How to avoid:**
- Apply a strict placement checklist before adding to `shared/`: Does this have business domain knowledge? If yes, it belongs in `entities/` or `features/`. Does it depend on a specific slice? If yes, it cannot be in `shared/`.
- Separate segment index files in `shared/ui` and `shared/lib` instead of one barrel — this forces conscious decisions about what is exported.
- Run `steiger` (FSD's structural linter) in CI to catch violations before merge.
- Concrete: a PDF metadata parser belongs in `entities/pdf-document/`, not `shared/lib/pdf/`. A "format bytes to human-readable string" utility belongs in `shared/lib/format/`.

**Warning signs:**
- `shared/` directory grows faster than any other layer
- You find yourself writing `import { PDFDocument } from '@/shared/types'` — a business type in shared
- `shared/api` contains request functions that reference a specific feature's data shape

**Phase to address:** Phase 1 (project scaffolding). Set up the layer structure and Steiger CI check before any feature code is written. This cannot be refactored cheaply later.

**Severity:** CRITICAL

---

### Pitfall 2: FSD Import Rule — Peer-Slice and Upward Imports

**What goes wrong:**
A `features/pdf-viewer/` slice imports directly from `features/widget-panel/` to sync state. A `widgets/dashboard/` imports from `pages/dashboard/` to access route params. Both are hard violations of FSD's unidirectional dependency rule (each layer may only import from layers below it; slices on the same layer cannot import each other).

**Why it happens:**
Convenience. When two slices need to coordinate — e.g., the PDF viewer needs to know which widgets are selected — the fastest fix is a direct import. Teams discover this pattern breaks FSD only when the first refactor propagates changes across six files that "shouldn't have known" about each other.

**How to avoid:**
- For cross-slice coordination, route through a lower layer: lift shared state into `entities/` (e.g., `entities/widget-selection/`) or `shared/` (for truly cross-cutting signals).
- For entity-to-entity cross-references (e.g., `entities/pdf-document` needs a type from `entities/widget`), use FSD's `@x` notation: `entities/pdf-document/@x/widget` creates an explicit, reviewable cross-import surface.
- Install `eslint-plugin-import` + `@feature-sliced/eslint-config` and fail CI on import violations.
- Concrete for this project: the widget-PDF page sync state (which widget corresponds to which PDF page) belongs in `entities/sync-state/` or `features/widget-page-sync/`, not as a direct import between the viewer and panel slices.

**Warning signs:**
- ESLint import-plugin errors on relative paths crossing slice directories
- A widget feature slice has an import path like `../../features/pdf-viewer/model`
- Adding a new feature requires touching 3+ unrelated slices

**Phase to address:** Phase 1 (ESLint config) and every feature phase (enforce on each PR).

**Severity:** CRITICAL

---

### Pitfall 3: pdf.js Worker Leak — Multiple `pdf.worker.js` Instances

**What goes wrong:**
Each time the React component tree mounts a PDF viewer (route navigation, hot reload, component re-mount), a new `pdf.worker.js` Web Worker is spawned. The old worker is never terminated. In SPAs, after navigating away and back to a PDF route several times, the browser runs 5–10 workers simultaneously. Memory climbs. Tab crashes.

**Why it happens:**
`react-pdf` and raw `pdfjs-dist` both require configuring `GlobalWorkerOptions.workerSrc`. If this is set inside a component or effect rather than once at module scope (typically in `app/providers` or `shared/lib/pdf/`), each mount creates a fresh worker registration.

Additionally, the `loadingTask` returned by `pdfjsLib.getDocument()` must be explicitly destroyed on unmount. Without calling `loadingTask.destroy()`, the worker thread retains references and is never garbage collected.

**How to avoid:**
- Configure `GlobalWorkerOptions.workerSrc` once, at application initialization level (`app/providers/pdf-provider.tsx`), never inside a component.
- In every `useEffect` that calls `pdfjsLib.getDocument()`, return a cleanup function that calls `loadingTask.destroy()`.
- Use a ref to hold the current `loadingTask` so the cleanup captures the correct instance.
- Concrete implementation pattern:
  ```typescript
  // app/providers/pdf-provider.tsx — run ONCE at app init
  import { GlobalWorkerOptions } from 'pdfjs-dist';
  GlobalWorkerOptions.workerSrc = new URL(
    'pdfjs-dist/build/pdf.worker.min.mjs',
    import.meta.url,
  ).toString();
  ```
  ```typescript
  // features/pdf-viewer/ui/PdfCanvas.tsx
  useEffect(() => {
    const task = pdfjsLib.getDocument(url);
    // ... render
    return () => { task.destroy(); };
  }, [url]);
  ```

**Warning signs:**
- Chrome Task Manager shows multiple `pdf.worker.js` entries growing with navigation
- Browser tab memory climbs monotonically when switching between PDF routes
- Heap snapshots show `PDFDocumentProxy` objects retained after component unmount

**Phase to address:** Phase 2 (PDF viewer implementation). Set up the worker configuration in Phase 1 scaffolding so it's never done incorrectly in feature phases.

**Severity:** CRITICAL

---

### Pitfall 4: Rendering All PDF Pages at Once — No Virtualization

**What goes wrong:**
The dashboard loads a 200-page PDF report. Without virtualization, `react-pdf` renders all 200 `<Page>` components immediately. Each page creates a canvas element and triggers a pdf.js render task. The browser allocates hundreds of MB of canvas memory, the tab freezes for 3–5 seconds on load, and scrolling becomes janky (sub-10 FPS).

**Why it happens:**
`react-pdf`'s `<Document>` API makes it trivially easy to render pages in a loop: `pages.map(n => <Page pageNumber={n} />)`. This works fine for 5-page documents and fails catastrophically for 50+ pages. The failure is not obvious during development with small test PDFs.

**How to avoid:**
- Use `@tanstack/virtual` (already in the stack) to virtualize the page list — only render pages within ±2 of the current viewport position.
- Pre-calculate page heights using `pdfDoc.getPage(n).then(p => p.getViewport({ scale: 1 }))` before rendering, so the virtualizer can allocate the correct scroll height without rendering all pages.
- Consider using `react-pdf-headless` (built on `react-pdf` + `@tanstack/virtual`) as a reference implementation.
- Never test PDF rendering performance with short documents — always test with the largest realistic PDF (aim for 100+ pages).
- pdf.js itself recommends not rendering more than 25 pages simultaneously.

**Warning signs:**
- PDF viewer component renders a loop of `<Page>` without an intersection observer or virtualizer
- Performance profiler shows canvas creation taking >500ms on load
- Memory usage exceeds 300MB for a single document view

**Phase to address:** Phase 2 (PDF viewer). Virtualization must be designed in from the start — retrofitting it after the viewer is built requires significant restructuring of the scroll container.

**Severity:** CRITICAL

---

### Pitfall 5: Zustand Storing Server/Remote State — Wrong Tool

**What goes wrong:**
The team puts PDF report metadata, widget data fetched from an API, and dashboard analytics into Zustand stores. Zustand has no built-in cache invalidation, background refresh, loading/error states, or stale-while-revalidate semantics. The implementation grows to manually track `isLoading`, `error`, `lastFetched` flags in each store slice. Cache invalidation is forgotten on mutations. Data goes stale silently.

**Why it happens:**
Zustand is simple and flexible. Because it _can_ hold anything, it's tempting to use it for everything. The team reaches for it before the TanStack Query integration is solid. The split between "UI state" and "server state" is not enforced architecturally.

**How to avoid:**
- **Rule:** Zustand holds UI state only — panel open/closed, current zoom level, selected widget ID, PDF scroll position, active filter selections.
- **Rule:** TanStack Query owns all server state — report metadata, widget definitions, analytics data, user preferences fetched from an API.
- When v1 uses mocks, still use TanStack Query with mock query functions. This ensures the same caching and invalidation patterns are in place when the real API is wired.
- Draw this boundary in the `app/providers/` architecture review before any feature code ships.

**Warning signs:**
- Zustand store contains properties named `isLoading`, `error`, `data`, `lastUpdated`
- An effect in a store action calls `fetch()` directly
- Mock data is imported directly into a Zustand store's `initialState`

**Phase to address:** Phase 1 (state management architecture). Establish the Zustand/TanStack Query boundary before any data flows are implemented.

**Severity:** CRITICAL

---

### Pitfall 6: Mock Data Hardcoded Into Components — Unmigrateable Mocks

**What goes wrong:**
v1 is scaffolded with mock data. The team imports mock JSON files directly into components (`import mockReports from '../mocks/reports.json'`), or hardcodes data in component bodies. When the real API is ready, every component must be individually modified. The diff is large, risky, and easy to miss.

**Why it happens:**
It's the fastest way to get data on screen. The team plans to "fix it later" but later never comes cleanly because components have grown around the mock data shape.

**How to avoid:**
- Structure mocks as TanStack Query query functions from day one:
  ```typescript
  // shared/api/reports/queries.ts
  export const reportQueries = {
    list: () => queryOptions({
      queryKey: ['reports'],
      queryFn: () => fetchReports(), // swap this function, never the component
    }),
  };

  // shared/api/reports/__mocks__/fetchReports.ts  (used in dev only)
  export const fetchReports = async () => mockReportData;
  ```
- Use MSW (Mock Service Worker) for network-level mocking. MSW intercepts actual HTTP requests, so when the real API ships, you remove MSW handlers — components and query functions change zero lines.
- Never import mock JSON directly into a component or a Zustand store initial state.
- Keep mock data in `shared/api/[domain]/__mocks__/` and ensure it exactly matches the shape of what the real API will return.

**Warning signs:**
- `import mockData from './data.json'` inside a component file
- Zustand `initialState` contains hardcoded arrays of objects
- Mock data lives in the same directory as the component that uses it

**Phase to address:** Phase 1 (mock infrastructure setup). The mock strategy must be decided before any feature data flows are built.

**Severity:** CRITICAL

---

## Moderate Pitfalls

### Pitfall 7: Zustand Selector Without `useShallow` — Excessive Re-renders

**What goes wrong:**
A component subscribes to multiple Zustand values using an object or array selector: `useStore(s => ({ zoom: s.zoom, page: s.currentPage }))`. Zustand compares the returned object by reference. Because a new object literal is created every render cycle, the component re-renders on every store update regardless of whether `zoom` or `currentPage` changed. In a PDF viewer with frequent scroll events updating position state, this causes cascading re-renders across every panel component.

**Why it happens:**
Developers coming from React Context or Redux `useSelector` (which uses shallow equality by default) assume Zustand does the same. It does not — Zustand uses strict reference equality unless you opt in.

**How to avoid:**
- Always use `useShallow` when selecting multiple values:
  ```typescript
  import { useShallow } from 'zustand/react/shallow';
  const { zoom, currentPage } = usePdfStore(
    useShallow(s => ({ zoom: s.zoom, currentPage: s.currentPage }))
  );
  ```
- For single primitive values, direct selection is fine: `const zoom = usePdfStore(s => s.zoom)`.
- For computed/derived values that create new references each call, memoize with `useMemo` outside the selector.
- Note: `useShallow` only does shallow (one-level) equality. Nested objects still re-render if the nested reference changes.

**Warning signs:**
- React DevTools Profiler shows components rendering on every mouse move in the PDF viewer
- Selector functions return object literals: `s => ({ a: s.a, b: s.b })`
- `useShallow` is not imported anywhere in the codebase

**Phase to address:** Phase 2+ (any phase implementing Zustand selectors).

**Severity:** HIGH

---

### Pitfall 8: TanStack Router `loaderDeps` Returning Full Search Object

**What goes wrong:**
A dashboard route has search params for filter state, sort order, and active PDF page. The route loader fetches report data. If `loaderDeps` returns the full search object (`loaderDeps: ({ search }) => search`), the loader re-runs on every search param change — including UI-only params like `sortDirection` that don't affect the API call. Every user interaction triggers a network request.

**Why it happens:**
The full search object spread is the "easy" pattern shown in introductory examples. Developers don't realize that `staleTime` defaults to `0ms` for navigations, meaning every navigation re-runs the loader.

**How to avoid:**
- Extract only the params that affect the data fetch:
  ```typescript
  loaderDeps: ({ search }) => ({ reportId: search.reportId, dateRange: search.dateRange }),
  loader: ({ deps }) => queryClient.ensureQueryData(reportQueries.detail(deps)),
  ```
- UI-only params (sort direction, panel width, zoom level) should live in Zustand, not route search params — they don't need to be in the URL unless deep-linking to that specific UI state is a requirement.
- Use `@tanstack/eslint-plugin-router` to catch common misconfiguration at lint time.

**Warning signs:**
- Network tab shows API requests firing on every sort/filter interaction
- `loaderDeps` function returns `search` directly without destructuring
- Route search params include UI state like `isLeftPanelOpen`

**Phase to address:** Phase 1 (routing setup) and Phase 3+ (route-data integration).

**Severity:** HIGH

---

### Pitfall 9: FSD "God Widget" — Business Logic in Widget Layer

**What goes wrong:**
The `widgets/dashboard-layout/` slice accumulates business logic: it fetches data directly, contains derived state calculations, manages widget-PDF sync state, and orchestrates cross-feature interactions. It becomes a 1000-line monolith that other developers are afraid to touch.

**Why it happens:**
Widgets are the natural place for "putting things together," and that responsibility easily expands to include business logic. FSD's widget layer is for composition — assembling features and entities into coherent UI blocks — not for owning business rules.

**How to avoid:**
- Widget slices should contain only: layout structure, composition of feature/entity components, and wiring of event handlers to feature model functions.
- Business logic belongs in `features/[name]/model/` (user-initiated actions, state machines) or `entities/[name]/model/` (domain data operations).
- A widget should be replaceable with a different widget without changing any business behavior.
- Apply the test: "If I deleted this widget and built a different layout, would any business rule be lost?" If yes, the logic is in the wrong place.

**Warning signs:**
- Widget files contain `useQuery`, `useMutation`, or complex `useEffect` chains with business rules
- Widget model files have 200+ lines
- Feature or entity slices import from a widget slice

**Phase to address:** Phase 2-3 (widget implementation). Establish clear widget composition patterns before multiple widgets are built.

**Severity:** HIGH

---

### Pitfall 10: TypeScript Barrel File Circular Imports

**What goes wrong:**
`entities/pdf-document/index.ts` exports `PDFDocument` and `PDFPage`. `PDFPage` internally imports `PDFDocument` via the barrel file (`import { PDFDocument } from '.'`) instead of a relative internal import. TypeScript compiles it, but at runtime Vite's module resolution hits a circular dependency. The export is `undefined`. The component crashes with a cryptic "cannot read properties of undefined" error that looks like a React rendering bug.

**Why it happens:**
Within a slice, developers sometimes use the slice's own public API index for imports (following the "always import through index" mental model). This is correct for external consumers but wrong for internal modules within the same slice.

**How to avoid:**
- Inside a slice, always use relative imports between internal files: `import { PDFDocument } from '../document'` not `import { PDFDocument } from '.'`.
- The `index.ts` barrel is only for external consumers of the slice.
- Avoid `export * from './ui/Thing'` wildcard re-exports in index files — list exports explicitly.
- Configure `eslint-plugin-import` with `no-cycle` rule to detect circular imports in CI.

**Warning signs:**
- Runtime error: "Cannot read properties of undefined" immediately on import
- `import { X } from '.'` or `import { X } from '..'` inside the same slice's internal files
- `export *` (wildcard) patterns in index files

**Phase to address:** Phase 1 (project setup) — configure ESLint `no-cycle` rule from the start.

**Severity:** HIGH

---

### Pitfall 11: Tailwind Class Sprawl Without Design Token Discipline

**What goes wrong:**
Without enforced design tokens, individual developers reach for arbitrary Tailwind values: `bg-[#2a3c4e]`, `p-[13px]`, `text-[0.85rem]`. After 3 months of parallel development, the dashboard has 40+ unique colors, 15 spacing values, and no consistent visual language. Theming (dark mode, customer white-labeling) becomes impossible.

**Why it happens:**
Tailwind makes arbitrary values easy (`[]` escape hatch). Without a configured theme, the "pit of success" becomes a pit of arbitrary values. Developers prioritize shipping over token discipline.

**How to avoid:**
- Define all colors, spacing, typography, and shadow values in `tailwind.config.ts` (or `@theme` in Tailwind v4) before writing any component styles. Only these tokens should be used in components.
- No arbitrary values (`[]`) except for true one-offs like `calc()` expressions. Prefer extending the theme for reused values.
- Use semantic naming: `text-primary`, `bg-surface`, `border-subtle` — not `text-blue-600` in component files.
- In FSD context: the `shared/ui/` layer defines base primitives using design tokens. Widget and feature layers consume `shared/ui/` components. They should never re-implement base styles from scratch.

**Warning signs:**
- `grep -r '\[#' src/` returns more than a handful of results
- Tailwind `tailwind.config.ts` has an empty or minimal `theme.extend`
- Widget-layer components directly compose raw Tailwind primitives instead of `shared/ui` components

**Phase to address:** Phase 1 (design system setup) — configure the theme before any UI is built.

**Severity:** HIGH

---

## Minor Pitfalls

### Pitfall 12: TanStack Router Parent `beforeLoad` Blocking Children

**What goes wrong:**
An authentication guard in a parent route's `beforeLoad` makes a slow API call to validate the session. All child routes in the tree are blocked until this resolves. The dashboard feels slow to load even when the user is authenticated, because `beforeLoad` runs sequentially while `loader` runs in parallel.

**How to avoid:**
- Keep `beforeLoad` lightweight — it should only redirect, check synchronous conditions, or read from a pre-populated cache. Do not make API calls in `beforeLoad`.
- For auth validation: preload user session data in the root layout loader. Use `beforeLoad` only for the synchronous redirect check based on that pre-loaded data.
- Child route data loading belongs in `loader`, which runs in parallel across all active routes.

**Warning signs:**
- `beforeLoad` contains `await fetch(...)` or `await queryClient.fetchQuery(...)`
- All dashboard sub-routes have identical loading delays even with cached data

**Phase to address:** Phase 1 (routing setup).

**Severity:** MEDIUM

---

### Pitfall 13: PDF Canvas High-DPI Rendering — Blurry Text

**What goes wrong:**
The PDF viewer renders canvases at logical CSS pixels without accounting for `devicePixelRatio`. On Retina/HiDPI displays, all text in the PDF appears blurry because the canvas is rendered at 1x and scaled up by CSS.

**How to avoid:**
- Always scale the canvas by `window.devicePixelRatio`:
  ```typescript
  const dpr = window.devicePixelRatio || 1;
  canvas.width = viewport.width * dpr;
  canvas.height = viewport.height * dpr;
  canvas.style.width = `${viewport.width}px`;
  canvas.style.height = `${viewport.height}px`;
  context.scale(dpr, dpr);
  ```
- Handle `resize` events to re-render if the viewport or DPR changes (e.g., user moves window between displays).

**Warning signs:**
- PDF text looks blurry on MacOS or high-DPI monitors
- Canvas width/height set without multiplying by `devicePixelRatio`

**Phase to address:** Phase 2 (PDF viewer implementation).

**Severity:** MEDIUM

---

### Pitfall 14: MSW Query Parameter Matching Failure

**What goes wrong:**
A developer writes an MSW handler as `http.get('/api/reports?status=active', resolver)`. MSW never matches this because query parameters cannot be part of the route path in MSW handlers — they must be extracted from `request.url` inside the handler. The mock never fires; the real network request fails silently in development, or the fetch hangs with no error.

**How to avoid:**
- MSW handlers must not include query strings in the URL path: `http.get('/api/reports', resolver)`.
- Read query params inside the resolver: `new URL(request.url).searchParams.get('status')`.

**Warning signs:**
- MSW handler defined but requests pass through to the real network (or fail with 404)
- Handler URL path contains `?` characters

**Phase to address:** Phase 1 (mock infrastructure) — document this rule when setting up MSW handlers.

**Severity:** MEDIUM

---

### Pitfall 15: FSD Segment Overloading — Everything in `ui/`

**What goes wrong:**
All files within a feature slice go into the `ui/` segment because developers don't know where else to put them. Business logic, API calls, type definitions, and constants all end up in `ui/` files. The segment distinction loses meaning.

**How to avoid:**
- Enforce standard FSD segments: `ui/` (React components), `model/` (state, store, selectors), `api/` (request functions), `lib/` (pure utility functions specific to this slice), `config/` (constants, configuration).
- A component file that contains both JSX and `useQuery`/`useMutation` is a red flag — split the data-fetching concern into `model/` or `api/`.

**Warning signs:**
- `features/pdf-viewer/ui/pdfStore.ts` — store file inside `ui/` segment
- Feature slice has only one segment (`ui/`) with 15+ files

**Phase to address:** Phase 1 (project scaffolding). Document the segment structure for the team.

**Severity:** MEDIUM

---

## Technical Debt Patterns

| Shortcut | Immediate Benefit | Long-term Cost | When Acceptable |
|----------|-------------------|----------------|-----------------|
| Mock JSON imported directly in component | Zero setup time | Every component must change during API migration | Never — use MSW or query function mocks instead |
| All PDF pages rendered without virtualization | Simple implementation | Tab crash on 50+ page documents | Never for a document dashboard |
| `GlobalWorkerOptions` set inside component | Works immediately | Worker leak on every remount | Never — set once at app init |
| Skip `index.ts` public API for a slice | Faster development | Any internal refactor becomes a public breaking change | Never in FSD |
| Zustand for all state including server data | One mental model | Manual loading/error tracking, no cache invalidation | Never — use TanStack Query for server state |
| Tailwind arbitrary values `[...]` for colors | Pixel-perfect design | Untraceable design decisions, theming impossible | Only for `calc()` layout math |
| One `beforeLoad` call for all auth/data | Simple auth pattern | All child routes wait sequentially | Acceptable only if call is sub-50ms (cache hit) |

---

## Performance Traps

| Trap | Symptoms | Prevention | When It Breaks |
|------|----------|------------|----------------|
| All PDF pages rendered simultaneously | 3-5s freeze on load, 5GB memory | Virtualize: render ±2 pages from viewport | Any PDF >25 pages |
| Zustand object selector without `useShallow` | Full re-render on every store mutation | Wrap with `useShallow()` | Any store with >5 updates/sec (scroll, zoom) |
| pdf.worker spawned per mount | Memory grows with navigation | Configure `GlobalWorkerOptions` once at app init | After 3-4 navigations |
| Barrel file in `shared/ui` with heavy deps | Large initial bundle, slow dev server | Per-component segment indexes, no wildcard `export *` | When heavy deps (charts, syntax highlighting) added |
| TanStack Router `loaderDeps: s => s` | API called on every search param change | Return only data-relevant params from `loaderDeps` | Any route with multiple search params |
| Canvas rendered at logical pixels | Blurry text on Retina displays | Multiply dimensions by `devicePixelRatio` | All HiDPI displays |

---

## "Looks Done But Isn't" Checklist

- [ ] **PDF Worker:** `GlobalWorkerOptions.workerSrc` configured once in `app/` layer — verify no component sets it locally
- [ ] **PDF Cleanup:** Every `pdfjsLib.getDocument()` call has a corresponding `loadingTask.destroy()` in `useEffect` cleanup
- [ ] **PDF Virtualization:** PDF viewer only renders visible pages — verify with a 100+ page document
- [ ] **Mock Strategy:** All mock data flows through TanStack Query query functions or MSW handlers — no direct JSON imports in components
- [ ] **Zustand Selectors:** All multi-value selectors wrapped with `useShallow` — check with React DevTools Profiler
- [ ] **FSD Linter:** `steiger` or `@feature-sliced/eslint-config` running in CI — verify CI fails on import violations
- [ ] **Barrel Circularity:** `eslint-plugin-import` `no-cycle` rule enabled — verify it catches internal-to-own-index imports
- [ ] **Design Tokens:** All colors and spacing reference `tailwind.config.ts` theme values — `grep -r '\[#'` returns zero results
- [ ] **Route Deps:** `loaderDeps` extracts only data-relevant search params — verify network tab shows no unnecessary API calls on UI-only param changes
- [ ] **HiDPI Canvas:** PDF pages rendered sharply on a Retina display — visual check required, not automated

---

## Recovery Strategies

| Pitfall | Recovery Cost | Recovery Steps |
|---------|---------------|----------------|
| God shared layer | HIGH | Audit all `shared/` exports, classify each by layer (entity/feature/shared), move in batches, update all import paths |
| Direct mock imports in components | MEDIUM | Add TanStack Query + MSW infrastructure, replace imports with `useQuery` calls, verify behavior parity |
| No PDF virtualization | HIGH | Restructure scroll container to use `@tanstack/virtual`, implement page height pre-calculation, rebuild page rendering loop |
| Worker leak | LOW | Move `GlobalWorkerOptions` to app init, add `destroy()` to useEffect cleanups — isolated changes |
| Zustand selector re-renders | LOW | Add `useShallow` imports, wrap affected selectors — mechanical find-and-fix |
| Arbitrary Tailwind values | MEDIUM | Audit all `[]` usages, add to `tailwind.config.ts` theme, replace arbitrary with token class |
| Circular barrel imports | LOW-MEDIUM | Fix specific circular paths flagged by ESLint `no-cycle`, change internal imports to relative paths |

---

## Pitfall-to-Phase Mapping

| Pitfall | Prevention Phase | Verification |
|---------|------------------|--------------|
| God shared layer | Phase 1: Scaffolding | Steiger passes in CI; `shared/` audit shows only cross-cutting utilities |
| FSD import violations (peer-slice, upward) | Phase 1: ESLint setup | `@feature-sliced/eslint-config` errors cause CI failure |
| PDF worker leak | Phase 1: App init + Phase 2: PDF viewer | Chrome Task Manager shows single worker after 10 navigations |
| PDF no virtualization | Phase 2: PDF viewer | 100-page document loads in <2s; memory stays <200MB |
| Zustand storing server state | Phase 1: Architecture | No `fetch` calls in Zustand actions; no `isLoading` in any store |
| Mock data hardcoded in components | Phase 1: Mock infrastructure | `grep -r 'import.*mock.*json' src/` returns zero results |
| Zustand selector re-renders | Phase 2+: Feature implementation | React Profiler shows no unexpected renders on scroll/zoom |
| `loaderDeps` full search object | Phase 1: Routing setup | Network tab shows loaders only fire when data-relevant params change |
| FSD god widget | Phase 2-3: Widget implementation | Widget slices contain no `useQuery`/`useMutation`; no >200-line model files |
| TypeScript barrel circular import | Phase 1: ESLint setup | `eslint-plugin-import` `no-cycle` runs in CI |
| Tailwind class sprawl | Phase 1: Design system | `grep -r '\[#' src/` returns zero results; theme configured before UI built |
| MSW query param matching | Phase 1: Mock infrastructure | Mock handlers match all intended requests; no pass-throughs to real network |
| Parent `beforeLoad` blocking children | Phase 1: Routing setup | Auth check resolves from cache in <10ms |
| HiDPI blurry canvas | Phase 2: PDF viewer | Visual check on Retina display passes |
| FSD segment overloading | Phase 1: Scaffolding | No `.ts` store/model files inside `ui/` segments |

---

## Sources

- [Feature-Sliced Design — Layers Reference](https://feature-sliced.design/docs/reference/layers) (official docs)
- [Feature-Sliced Design — Public API Reference](https://feature-sliced.design/docs/reference/public-api) (official docs)
- [ESLint Plugin FSD Imports](https://socket.dev/npm/package/eslint-plugin-feature-sliced-design-imports) (MEDIUM confidence — WebSearch)
- [react-pdf — Memory Leak Issue #504](https://github.com/wojtekmaj/react-pdf/issues/504) (GitHub issue — MEDIUM confidence)
- [react-pdf — Large PDF Performance Discussion #1691](https://github.com/wojtekmaj/react-pdf/discussions/1691) (GitHub discussion — MEDIUM confidence)
- [pdf.js loadingTask.destroy() memory leak](https://pdfjs.community/t/huge-memory-leak-react/2577) (community forum — MEDIUM confidence)
- [Zustand useShallow docs](https://zustand.docs.pmnd.rs/guides/prevent-rerenders-with-use-shallow) (official docs — HIGH confidence)
- [Zustand re-renders discussion #2642](https://github.com/pmndrs/zustand/discussions/2642) (GitHub — MEDIUM confidence)
- [TanStack Router Data Loading](https://tanstack.com/router/v1/docs/framework/react/guide/data-loading) (official docs — HIGH confidence)
- [TanStack Router in production — Swizec Teller](https://swizec.com/blog/tips-from-8-months-of-tan-stack-router-in-production/) (practitioner blog — MEDIUM confidence)
- [MSW FAQ — Query parameter matching](https://mswjs.io/docs/faq/) (official docs — HIGH confidence)
- [TanStack Query + MSW cache pitfalls](https://github.com/TanStack/query/discussions/6455) (GitHub — MEDIUM confidence)
- [Tailwind CSS best practices 2025](https://www.frontendtools.tech/blog/tailwind-css-best-practices-design-system-patterns) (WebSearch — LOW confidence, corroborated by official Tailwind docs)
- [FSD steiger linter](https://github.com/feature-sliced/steiger) (official FSD tooling — HIGH confidence)
- [FSD @x notation for cross-entity imports](https://feature-sliced.design/docs/reference/public-api) (official docs — HIGH confidence)
- [react-virtualized-pdf](https://www.npmjs.com/package/react-virtualized-pdf) (npm — MEDIUM confidence)

---
*Pitfalls research for: React PDF Analytics Dashboard with Feature Sliced Design*
*Researched: 2026-02-18*
