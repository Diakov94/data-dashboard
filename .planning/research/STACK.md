# Stack Research

**Domain:** PDF Analytics Dashboard with Feature-Sliced Design (FSD)
**Researched:** 2026-02-18
**Confidence:** HIGH (versions verified via WebSearch against official npm/GitHub release pages; FSD patterns verified against official FSD docs; integration patterns verified via multiple authoritative sources)

---

## Recommended Stack

### Core Technologies

| Technology | Version | Purpose | Why Recommended |
|------------|---------|---------|-----------------|
| React | 19.2.4 | UI framework | Latest stable; Actions API, Activity component, native metadata. React 19 is production-ready and all chosen libraries are compatible. |
| TypeScript | 5.9.3 | Type safety | Latest stable on npm. `strict: true` enforced from day 1 — FSD's strict import boundaries are only reliably enforced when TypeScript can see all cross-slice imports. |
| Vite | 7.x (latest) | Build tool & dev server | Fastest cold starts via native ESM, HMR in microseconds. The `@tailwindcss/vite` plugin + `vite-tsconfig-paths` plugin integrate cleanly. Scaffolds with `react-ts` template. |
| @vitejs/plugin-react | 5.1.4 | JSX/TSX transform for Vite | Official Vite React plugin; handles SWC-based fast refresh automatically. |

### State Management

| Technology | Version | Purpose | Why Recommended |
|------------|---------|---------|-----------------|
| Zustand | 5.0.11 | Client-side UI state | Minimal boilerplate; drops `use-sync-external-store` dependency by using native React 18+ `useSyncExternalStore`. Per-slice store pattern maps 1:1 to FSD's `model` segment — each slice owns one store. Never use for server state (that is TanStack Query's job). |
| @tanstack/react-query | 5.90.20 | Server state (fetch/cache/sync) | v5 unified API (`queryOptions` pattern). Handles PDF list, report metadata, upload progress. Lives in `entities/*/api` and `features/*/api` segments. Reduces Zustand surface area to UI state only. |

### Routing

| Technology | Version | Purpose | Why Recommended |
|------------|---------|---------|-----------------|
| @tanstack/react-router | 1.160.x (latest) | Client-side routing | Fully type-safe route params and search params. From the same team as TanStack Query — designed to work together. Code-based routing fits FSD perfectly: route tree lives in `app/`, page components live in `pages/`. Avoids the file-based routing conflict with FSD's `pages/` layer convention. |

### Styling

| Technology | Version | Purpose | Why Recommended |
|------------|---------|---------|-----------------|
| tailwindcss | 4.1.18 | Utility-first CSS | v4 is a ground-up rewrite: no `tailwind.config.js` required, auto-detects content files, 5x faster full builds, 100x faster incremental builds. Single `@import "tailwindcss"` in the CSS entry. Use `@tailwindcss/vite` plugin (NOT PostCSS) for Vite projects. Lives in `shared/ui` styles; utility classes applied directly in component JSX across all layers. |
| @tailwindcss/vite | 4.x | Vite plugin for Tailwind v4 | Required for Tailwind v4 in Vite projects. Eliminates PostCSS config entirely. |

### PDF Rendering

| Technology | Version | Purpose | Why Recommended |
|------------|---------|---------|-----------------|
| react-pdf (by wojtekmaj) | 10.3.0 | Render PDF files in React | Wraps PDF.js (`pdfjs-dist`) with idiomatic React components. Peer dep updated to `pdfjs-dist` 5.4.296. React 19 compatible. Open-source. Provides `<Document>` and `<Page>` primitives — the right level of abstraction for a custom viewer with virtualization + minimap + zoom. No prebuilt toolbar (intentional: we build our own). |

### Virtualization

| Technology | Version | Purpose | Why Recommended |
|------------|---------|---------|-----------------|
| @tanstack/react-virtual | 3.13.18 | Virtualize large PDF page lists | Headless — renders only visible pages, reducing memory footprint for 100+ page PDFs. Supports dynamic item sizes (critical: PDF pages vary in height). Uses `useVirtualizer` hook pattern that integrates cleanly into React components. Outperforms react-window on variable-height content and is from the same TanStack ecosystem. Set `useFlushSync: false` for React 19 compatibility. |

### Charts / Data Visualization

| Technology | Version | Purpose | Why Recommended |
|------------|---------|---------|-----------------|
| Recharts | latest (2.x) | Analytics charts and widgets | Composable, JSX-native API, SVG-based. Ideal for internal analytics dashboards. Gentlest learning curve for a small team. Batteries-included for bar, line, area, pie charts needed for report widgets. Lives in `widgets/` and `entities/` ui segments. |

### FSD Architecture Enforcement

| Technology | Version | Purpose | Why Recommended |
|------------|---------|---------|-----------------|
| steiger + @feature-sliced/steiger-plugin | latest | Architectural linting | Official FSD CLI linter. Catches violations (cross-layer imports, missing public APIs, wrong slice depth) automatically. Config inspired by ESLint — easy to adopt. Run in CI. |
| @feature-sliced/eslint-config | latest | ESLint FSD rules | Uses `eslint-plugin-boundaries` to enforce FSD import rules at the editor level (real-time feedback). Pair with Steiger for dual coverage (editor + CI). |
| vite-tsconfig-paths | 5.x | Sync tsconfig path aliases into Vite | Reads tsconfig `paths` once; Vite picks them up automatically. Eliminates dual-maintenance of aliases in both `tsconfig.json` and `vite.config.ts`. CSS imports not resolved (known limitation), but TypeScript/JS aliases work perfectly. |

### Development Tools

| Tool | Purpose | Notes |
|------|---------|-------|
| ESLint | Linting | With `@feature-sliced/eslint-config` + `eslint-plugin-boundaries`; enforce FSD import rules |
| Prettier | Code formatting | Pair with ESLint; standard for small teams |
| TypeScript strict mode | Type safety | `strict: true` in tsconfig; required for TanStack Router's full type inference |
| @tanstack/router-devtools | Route/search param debugging | Include dev-only; helps debug type-safe route tree |
| @tanstack/react-query-devtools | Query cache debugging | Include dev-only; shows cache state, stale times |

---

## FSD Layer Mapping

This section is the primary output for FSD-aware roadmap planning.

### Directory Structure

```
src/
├── app/                    # App layer — bootstrapping only
│   ├── router.tsx          # createRootRoute() + createRouter() + RouterProvider
│   ├── providers.tsx       # QueryClient, global providers
│   ├── styles/
│   │   └── index.css       # @import "tailwindcss" entry point
│   └── index.tsx           # App root
│
├── pages/                  # Pages layer — one slice per route
│   ├── reports-list/       # /  (root route)
│   │   ├── ui/
│   │   │   └── ReportsListPage.tsx
│   │   ├── api/
│   │   │   └── reportsListQuery.ts
│   │   └── index.ts        # public API
│   └── report-view/        # /reports/:reportId
│       ├── ui/
│       │   └── ReportViewPage.tsx
│       ├── api/
│       │   └── reportQuery.ts
│       ├── model/
│       │   └── reportViewStore.ts   # Zustand store for viewer state
│       └── index.ts
│
├── widgets/                # Widgets layer — self-contained UI blocks
│   ├── pdf-viewer/         # Right panel: virtualized PDF viewer
│   │   ├── ui/
│   │   ├── model/
│   │   └── index.ts
│   ├── report-widgets-panel/  # Left panel: charts/tables/stats
│   │   ├── ui/
│   │   └── index.ts
│   └── upload-modal/
│       ├── ui/
│       ├── model/          # upload state store
│       └── index.ts
│
├── features/               # Features layer — user actions
│   ├── upload-pdf/
│   │   ├── ui/
│   │   ├── api/            # POST /reports mutation
│   │   └── index.ts
│   ├── zoom-pdf/
│   │   ├── model/          # zoom level store slice
│   │   └── index.ts
│   └── sync-page-widget/   # Widget-page synchronization feature
│       ├── model/
│       └── index.ts
│
├── entities/               # Entities layer — business objects
│   ├── report/
│   │   ├── api/
│   │   │   ├── reportApi.ts       # fetch functions
│   │   │   └── reportKeys.ts      # queryKey factory
│   │   ├── model/
│   │   │   └── reportStore.ts     # selected report state
│   │   ├── ui/
│   │   │   └── ReportCard.tsx
│   │   └── index.ts
│   └── pdf-page/
│       ├── ui/
│       │   └── PdfPageRenderer.tsx  # single page via react-pdf <Page>
│       └── index.ts
│
└── shared/                 # Shared layer — zero business logic
    ├── api/
    │   └── client.ts       # axios/fetch base client + interceptors
    ├── ui/                 # Reusable primitives
    │   ├── Button.tsx
    │   ├── Spinner.tsx
    │   └── ...
    ├── lib/
    │   └── queryClient.ts  # QueryClient singleton
    ├── routes/
    │   └── paths.ts        # Route path constants (string literals)
    └── config/
        └── env.ts          # Environment variable access
```

### Zustand Store Pattern in FSD (`model` segment)

Each FSD slice that needs client state owns a single Zustand store in its `model/` segment. The store is exported only via the slice's `index.ts` public API.

```typescript
// src/pages/report-view/model/reportViewStore.ts
import { create } from 'zustand'

interface ReportViewState {
  activeWidgetId: string | null
  currentPage: number
  zoomLevel: number
  setActiveWidget: (id: string | null) => void
  setCurrentPage: (page: number) => void
  setZoomLevel: (zoom: number) => void
}

export const useReportViewStore = create<ReportViewState>((set) => ({
  activeWidgetId: null,
  currentPage: 1,
  zoomLevel: 1.0,
  setActiveWidget: (id) => set({ activeWidgetId: id }),
  setCurrentPage: (page) => set({ currentPage: page }),
  setZoomLevel: (zoom) => set({ zoomLevel: zoom }),
}))
```

```typescript
// src/pages/report-view/index.ts  (public API)
export { ReportViewPage } from './ui/ReportViewPage'
export { useReportViewStore } from './model/reportViewStore'
// Do NOT export internal api/ or lib/ — keep them slice-private
```

**Rule:** Widgets and features that need viewer state import `useReportViewStore` from `@pages/report-view`, not from the internal path. This preserves FSD's public API contract.

**Cross-slice state coordination:** If two widgets need shared state (e.g., current PDF page drives both the viewer and the widget panel), lift that state to the `widgets/` or `features/` layer that owns the coordination, not into `shared/`. Avoid a global "app store" — it defeats FSD slice isolation.

### TanStack Router + FSD Integration Pattern

**Use code-based routing.** File-based routing expects `src/routes/` which conflicts with FSD's `src/pages/`. Code-based routing keeps the route tree in `app/router.tsx` and page components in `pages/`.

```typescript
// src/app/router.tsx
import { createRootRoute, createRoute, createRouter } from '@tanstack/react-router'
import { ReportsListPage } from '@pages/reports-list'
import { ReportViewPage } from '@pages/report-view'

const rootRoute = createRootRoute({
  component: RootLayout,   // global shell with nav
})

const reportsListRoute = createRoute({
  getParentRoute: () => rootRoute,
  path: '/',
  component: ReportsListPage,
})

const reportViewRoute = createRoute({
  getParentRoute: () => rootRoute,
  path: '/reports/$reportId',
  component: ReportViewPage,
  validateSearch: (search) => ({
    widgetId: search.widgetId as string | undefined,
  }),
})

const routeTree = rootRoute.addChildren([reportsListRoute, reportViewRoute])

export const router = createRouter({ routeTree })

// Required for full type safety
declare module '@tanstack/react-router' {
  interface Register {
    router: typeof router
  }
}
```

```typescript
// src/app/index.tsx
import { RouterProvider } from '@tanstack/react-router'
import { router } from './router'

export function App() {
  return <RouterProvider router={router} />
}
```

**URL-based widget sublinks:** The `widgetId` search param (`/reports/123?widgetId=chart-revenue`) is validated in the route definition via `validateSearch`. Page components read it with `useSearch()` — fully type-safe, no manual URL parsing.

### TanStack Query queryKey Pattern in FSD

**Rule:** `queryKey` factories live in the `api/` segment of the entity that owns the data. Query hooks (wrapping `useQuery`) live in the same `api/` segment and are exported via `index.ts`.

```typescript
// src/entities/report/api/reportKeys.ts
export const reportKeys = {
  all: ['reports'] as const,
  lists: () => [...reportKeys.all, 'list'] as const,
  list: (filters?: ReportFilters) => [...reportKeys.lists(), { filters }] as const,
  detail: (reportId: string) => [...reportKeys.all, 'detail', reportId] as const,
}

// src/entities/report/api/reportApi.ts
import { queryOptions } from '@tanstack/react-query'
import { reportKeys } from './reportKeys'

export const reportListQuery = (filters?: ReportFilters) =>
  queryOptions({
    queryKey: reportKeys.list(filters),
    queryFn: () => fetchReports(filters),
    staleTime: 5 * 60 * 1000,
  })

export const reportDetailQuery = (reportId: string) =>
  queryOptions({
    queryKey: reportKeys.detail(reportId),
    queryFn: () => fetchReport(reportId),
    staleTime: 10 * 60 * 1000,
  })
```

```typescript
// Consumer in pages/reports-list/api/reportsListQuery.ts
import { useQuery } from '@tanstack/react-query'
import { reportListQuery } from '@entities/report'

export const useReportsListQuery = (filters?: ReportFilters) =>
  useQuery(reportListQuery(filters))
```

**`shared/api` scope:** Only the base HTTP client and interceptors live in `shared/api/client.ts`. Business-domain query keys/functions never live in `shared/` — they live in the entity or feature that owns them.

### react-pdf Placement in FSD

`react-pdf` components are UI primitives that wrap PDF.js. The single-page renderer lives in `entities/pdf-page/ui/PdfPageRenderer.tsx`. The full virtualized viewer (multi-page, scroll, minimap) lives in `widgets/pdf-viewer/`.

```typescript
// src/entities/pdf-page/ui/PdfPageRenderer.tsx
import { Page } from 'react-pdf'

// Renders exactly one PDF page — no scroll, no state
export function PdfPageRenderer({ pageNumber, scale }: Props) {
  return <Page pageNumber={pageNumber} scale={scale} renderTextLayer={false} />
}
```

```typescript
// src/widgets/pdf-viewer/ui/PdfViewer.tsx
import { Document } from 'react-pdf'
import { useVirtualizer } from '@tanstack/react-virtual'
import { PdfPageRenderer } from '@entities/pdf-page'

// Combines Document + virtualizer + PdfPageRenderer
// Worker setup happens once at app level in shared/lib or app/
```

**PDF.js Worker setup (once, at app level):**
```typescript
// src/app/index.tsx (or shared/lib/pdfWorker.ts imported once)
import { pdfjs } from 'react-pdf'
pdfjs.GlobalWorkerOptions.workerSrc = new URL(
  'pdfjs-dist/build/pdf.worker.min.mjs',
  import.meta.url,
).toString()
```

### Vite + TypeScript Path Aliases for FSD

**Recommended approach: `vite-tsconfig-paths` plugin** — define aliases once in `tsconfig.json`, Vite picks them up automatically.

```json
// tsconfig.app.json
{
  "compilerOptions": {
    "target": "ES2022",
    "lib": ["ES2022", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "moduleResolution": "bundler",
    "jsx": "react-jsx",
    "strict": true,
    "baseUrl": ".",
    "paths": {
      "@app/*":      ["./src/app/*"],
      "@pages/*":    ["./src/pages/*"],
      "@widgets/*":  ["./src/widgets/*"],
      "@features/*": ["./src/features/*"],
      "@entities/*": ["./src/entities/*"],
      "@shared/*":   ["./src/shared/*"]
    }
  },
  "include": ["src"]
}
```

```typescript
// vite.config.ts
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import tailwindcss from '@tailwindcss/vite'
import tsconfigPaths from 'vite-tsconfig-paths'

export default defineConfig({
  plugins: [
    react(),
    tailwindcss(),
    tsconfigPaths(),  // reads tsconfig paths automatically
  ],
})
```

**No manual alias duplication required.** `vite-tsconfig-paths` v5/v6 supports `projectDiscovery: 'lazy'` and auto-reloads on tsconfig change. Known limitation: CSS `@import` aliases are not resolved by this plugin (use relative CSS paths or `@layer` in your `index.css`).

---

## Installation

```bash
# Create project
npm create vite@latest data-dashboard -- --template react-ts
cd data-dashboard

# Core React ecosystem (React 19 already included by Vite template, upgrade if needed)
npm install react@latest react-dom@latest

# Routing + State + Data
npm install @tanstack/react-router @tanstack/react-query zustand

# PDF Rendering
npm install react-pdf

# Virtualization
npm install @tanstack/react-virtual

# Charts
npm install recharts

# Tailwind CSS (v4)
npm install -D tailwindcss @tailwindcss/vite

# FSD path aliases
npm install -D vite-tsconfig-paths

# FSD enforcement
npm install -D steiger @feature-sliced/steiger-plugin
npm install -D @feature-sliced/eslint-config eslint-plugin-import eslint-plugin-boundaries

# TypeScript (ensure latest)
npm install -D typescript@latest
```

---

## Alternatives Considered

| Recommended | Alternative | When to Use Alternative |
|-------------|-------------|-------------------------|
| @tanstack/react-router | React Router v6/v7 | If team already knows RRv6 deeply and can't absorb a new routing paradigm. RR lacks built-in search param validation and is less type-safe. |
| @tanstack/react-router | Next.js App Router | Only if SSR/RSC become a firm requirement. Next.js file-based routing conflicts fundamentally with FSD's `pages/` layer — requires workarounds. |
| Zustand | Redux Toolkit | Only for teams with existing RTK infrastructure or needing time-travel debugging. RTK is heavier and has more boilerplate for a small team. |
| @tanstack/react-query | SWR | SWR is simpler but has fewer features (no mutation lifecycle, no optimistic updates out of box, smaller ecosystem). Query v5 is the better fit for a dashboard with complex data. |
| react-pdf (wojtekmaj) | pdfjs-dist direct | Direct use of pdfjs-dist gives maximum control but requires manual React integration, canvas management, and worker setup. react-pdf's `<Document>/<Page>` saves 2-4 days of integration work. |
| react-pdf (wojtekmaj) | Nutrient (PSPDFKit) | Nutrient is commercial, provides prebuilt toolbar/annotations/forms. Use only if annotation/form-filling features are v1 requirements (they are not for this dashboard). |
| @tanstack/react-virtual | react-window | react-window is simpler but does not support variable-height items out of the box. PDF pages vary in height — react-virtual handles this with dynamic measurement. |
| @tanstack/react-virtual | react-virtualized | react-virtualized is larger and older; its successor is react-window. @tanstack/react-virtual is actively maintained and ecosystem-aligned. |
| Recharts | Visx (by Airbnb) | Use Visx if the team has D3 expertise and needs pixel-perfect custom charts. Visx has a steep learning curve — wrong choice for a v1 dashboard on a small team. |
| Recharts | Victory | Victory excels at React Native cross-platform. This project is web-only. |
| Tailwind CSS v4 | Tailwind CSS v3 | Use v3 if browser support requirements include Safari < 16.4, Chrome < 111, or Firefox < 128. Otherwise v4 is strictly better. |
| Vite | Create React App | CRA is deprecated. Never use in 2025. |
| vite-tsconfig-paths | Manual alias duplication | Manual approach works but requires keeping `tsconfig.json` and `vite.config.ts` in sync — a maintenance burden with 6 FSD layer aliases. |
| Steiger + @feature-sliced/eslint-config | eslint-plugin-fsd-lint | `eslint-plugin-fsd-lint` is a good ESLint 9+ flat config alternative, but Steiger is the officially maintained FSD tool and provides broader architectural checks beyond just imports. |

---

## What NOT to Use

| Avoid | Why | Use Instead |
|-------|-----|-------------|
| Redux / Redux Toolkit | Overkill for a small team's client state; FSD's per-slice model maps badly to Redux's single global store convention | Zustand per-slice stores |
| MobX | Implicit mutations make FSD boundaries opaque — side effects are hard to trace across slices | Zustand with explicit actions |
| React Context for shared state | Context re-renders all consumers on any state change; breaks performance for PDF viewer state (frequent updates) | Zustand with selectors |
| react-router-dom v5 | End of life; no TypeScript-first design; search param handling is manual | @tanstack/react-router |
| file-based TanStack Router | `src/routes/` convention conflicts with FSD `src/pages/` — forces painful co-location hacks | Code-based routing in `app/router.tsx` |
| @react-pdf/renderer (diegomura) | This is for GENERATING PDFs, not displaying them. Common naming confusion. | react-pdf (wojtekmaj) for display |
| pdfjs-dist v3 or v4 | react-pdf 10.x peer dep is pdfjs-dist 5.x; mismatched versions cause worker errors | Install react-pdf 10.x which pulls in pdfjs-dist 5.x automatically |
| A global Zustand store | Defeats FSD's slice isolation; cross-slice state becomes implicit coupling | Lift shared state to the lowest common ancestor layer, or use a well-named feature slice for coordination |
| Barrel files that re-export everything | FSD requires public APIs to be deliberate; `export * from './ui'` breaks tree-shaking and exposes internals | Explicit named exports in each slice's `index.ts` |
| CSS Modules with Tailwind v4 | v4 auto-detects all files; mixing module scoping with utility classes adds unnecessary complexity | Use Tailwind utilities directly in JSX; use `@layer components` for complex repeated patterns |

---

## Stack Patterns by Variant

**If the report page needs URL-based widget deep-linking:**
- Use TanStack Router's `validateSearch` in the `reportViewRoute`
- Store `widgetId` as a search param, not in Zustand
- Because URL state is bookmarkable/shareable; component state is ephemeral

**If PDF upload progress needs real-time tracking:**
- Use Zustand in `features/upload-pdf/model/` for optimistic upload state
- Use TanStack Query mutations with `onProgress` for the actual HTTP upload
- Because the progress bar is UI-local state (Zustand), while the server confirmation is server state (Query)

**If the PDF viewer needs page-to-widget synchronization:**
- Use a `features/sync-page-widget/` slice with a Zustand store
- Both `widgets/pdf-viewer` and `widgets/report-widgets-panel` import from this feature's public API
- Because synchronization is a feature (a user-visible capability), not a shared utility

**If v1 uses mocked data (confirmed requirement):**
- Wire TanStack Query as normal but point `queryFn` at mock JSON files or MSW handlers in `shared/api/`
- Keep the same `queryKey` factories — switching to real API later is a one-line change per query function
- Because the query/cache layer works identically with mocks; no architectural rework needed at v2

**If auth is needed (confirmed requirement, small team):**
- Use TanStack Router's `beforeLoad` guard in the root route
- Store auth token in `shared/lib/auth.ts` (not in a slice — auth is cross-cutting)
- Use a dedicated Zustand store in `app/` or `shared/` for auth session state

---

## Version Compatibility

| Package | Compatible With | Notes |
|---------|-----------------|-------|
| react@19.2.x | react-dom@19.2.x | Always keep react + react-dom at identical versions |
| react@19.x | react-pdf@10.x | Confirmed compatible per npm metadata |
| react@19.x | @tanstack/react-router@1.x | Confirmed; React 19 is a supported peer dep |
| react@19.x | @tanstack/react-query@5.x | Confirmed; v5 supports React 18+ |
| react@19.x | zustand@5.x | Confirmed; v5 dropped support for React < 18, uses native `useSyncExternalStore` |
| react@19.x | @tanstack/react-virtual@3.x | Confirmed; set `useFlushSync: false` to suppress React 19 flushSync warning |
| react@19.x | recharts@2.x | Compatible; recharts 2.x supports React 18+ |
| tailwindcss@4.x | @tailwindcss/vite@4.x | Must match major versions; use `@tailwindcss/vite` NOT PostCSS plugin in Vite projects |
| tailwindcss@4.x | vite@7.x | Compatible; v4's Vite plugin targets Vite 5+ / 6+ / 7+ |
| vite-tsconfig-paths@5.x | vite@7.x | Compatible |
| pdfjs-dist@5.x | react-pdf@10.x | react-pdf 10.x ships `pdfjs-dist@5.x` as peer dep; do not manually install an older pdfjs-dist |

---

## Sources

- [Feature-Sliced Design official docs — slices/segments reference](https://feature-sliced.design/docs/reference/slices-segments) — HIGH confidence (official docs)
- [FSD + Zustand guide (official FSD blog)](https://feature-sliced.design/blog/zustand-simple-state-guide) — HIGH confidence (official)
- [TanStack Router — routing concepts and file-based routing docs](https://tanstack.com/router/v1/docs/framework/react/routing/routing-concepts) — HIGH confidence (official)
- [TanStack Router — code-based routing](https://tanstack.com/router/v1/docs/routing/code-based-routing) — HIGH confidence (official)
- [react-pdf GitHub (wojtekmaj)](https://github.com/wojtekmaj/react-pdf) — HIGH confidence (official)
- [react-pdf npm page — version 10.3.0](https://www.npmjs.com/package/react-pdf) — HIGH confidence (npm registry)
- [TanStack Virtual npm — version 3.13.18](https://www.npmjs.com/package/@tanstack/react-virtual) — HIGH confidence (npm registry)
- [TanStack Query npm — version 5.90.20](https://www.npmjs.com/package/@tanstack/react-query) — HIGH confidence (npm registry)
- [TanStack Router npm — version 1.160.x](https://www.npmjs.com/package/@tanstack/react-router) — HIGH confidence (npm registry)
- [Zustand npm — version 5.0.11](https://www.npmjs.com/package/zustand) — HIGH confidence (npm registry)
- [React 19.2 release blog](https://react.dev/blog/2025/10/01/react-19-2) — HIGH confidence (official)
- [Tailwind CSS v4.0 release blog](https://tailwindcss.com/blog/tailwindcss-v4) — HIGH confidence (official)
- [tailwindcss npm — version 4.1.18](https://www.npmjs.com/package/tailwindcss) — HIGH confidence (npm registry)
- [Vite npm — version 7.3.1](https://www.npmjs.com/package/vite) — HIGH confidence (npm registry)
- [TypeScript 5.8 announcement](https://devblogs.microsoft.com/typescript/announcing-typescript-5-8/) — HIGH confidence (official); 5.9.3 is current stable
- [Steiger — official FSD linter](https://github.com/feature-sliced/steiger) — HIGH confidence (official GitHub)
- [vite-tsconfig-paths GitHub](https://github.com/aleclarson/vite-tsconfig-paths) — HIGH confidence (official GitHub)
- [TanStack Query — queryOptions pattern in v5](https://tanstack.com/query/v5) — HIGH confidence (official)
- [TkDodo — Effective React Query Keys pattern](https://tkdodo.eu/blog/effective-react-query-keys) — MEDIUM confidence (widely cited community resource)
- [FSD + TanStack Query + Axios example (taras.one)](https://taras.one/blog/tanstack-react-query-request-factory-example-with-fsd-and-axios) — MEDIUM confidence (community, specific technical content)

---

*Stack research for: PDF Analytics Dashboard with Feature-Sliced Design*
*Researched: 2026-02-18*
