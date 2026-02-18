# Codebase Structure

**Analysis Date:** 2026-02-18

## Directory Layout

```
data-dashboard/
├── index.js                    # Application entry point
├── package.json                # Project metadata and dependencies
├── CLAUDE.md                   # Claude Code guidance
├── README.md                   # Project overview
├── .mcp.json                   # MCP server configuration
├── .planning/
│   └── codebase/               # Codebase documentation (generated)
├── src/                        # Source code (to be created)
│   ├── models/                 # Data models and types
│   ├── services/               # Business logic
│   ├── controllers/            # Request handlers (if HTTP)
│   ├── utils/                  # Utility functions
│   └── middleware/             # Express/HTTP middleware (if applicable)
├── tests/                      # Test files (to be created)
│   ├── unit/
│   ├── integration/
│   └── fixtures/
├── docs/                       # Documentation (to be created)
└── node_modules/               # Dependencies (installed)
```

## Directory Purposes

**Root:**
- Purpose: Project root, entry point location, configuration files
- Contains: `index.js`, `package.json`, configuration files
- Key files: `index.js` (application entry), `package.json` (dependencies)

**`src/`:**
- Purpose: All application source code
- Contains: Models, services, controllers, utilities
- Key files: Created during implementation

**`src/models/`:**
- Purpose: Data structure definitions and type definitions
- Contains: Report model, analytics result model, configuration schemas
- Key files: `Report.js`, `Analytics.js` (to be created)

**`src/services/`:**
- Purpose: Business logic and core functionality
- Contains: PDF parsing, analytics computation, data extraction
- Key files: `PDFParser.js`, `AnalyticsEngine.js` (to be created)

**`src/controllers/`:**
- Purpose: HTTP request/response handling (if HTTP API)
- Contains: Route handlers, request validation, response formatting
- Key files: `ReportController.js`, `AnalyticsController.js` (to be created)

**`src/utils/`:**
- Purpose: Shared utility functions and helpers
- Contains: File operations, formatting, validation helpers, constants
- Key files: `validators.js`, `formatters.js` (to be created)

**`tests/`:**
- Purpose: Automated test suite
- Contains: Unit tests, integration tests, test fixtures
- Key files: Created during implementation

**`docs/`:**
- Purpose: Project documentation (separate from CLAUDE.md)
- Contains: API documentation, setup guides, architecture diagrams
- Key files: To be created as needed

## Key File Locations

**Entry Points:**
- `index.js`: Main application entry point (root directory)
- `package.json`: Defines `main: "index.js"` - Node.js starts here

**Configuration:**
- `.mcp.json`: MCP server configuration (context7, sequential-thinking)
- `package.json`: NPM configuration, scripts, metadata
- `CLAUDE.md`: Development guidance for Claude Code

**Core Logic:**
- `src/services/`: Where business logic lives
- `src/models/`: Where data structures are defined
- `src/controllers/`: Where HTTP/CLI handling occurs (if applicable)

**Testing:**
- `tests/unit/`: Unit test files
- `tests/integration/`: Integration test files
- `tests/fixtures/`: Test data and mock PDFs

## Naming Conventions

**Files:**
- PascalCase for classes/models: `Report.js`, `AnalyticsEngine.js`
- camelCase for utilities/functions: `pdfParser.js`, `validators.js`
- kebab-case for configuration: `.mcp.json`, `package.json`
- `.test.js` or `.spec.js` suffix for test files

**Directories:**
- lowercase with underscores for logical groups: `src/`, `tests/`
- plural for collections: `src/models/`, `src/services/`
- descriptive single names: `fixtures/`, `utils/`

**Functions:**
- Verb-based names: `parsePDF()`, `extractData()`, `computeAnalytics()`
- Predicate functions with `is/has/should` prefix: `isValidPDF()`, `hasContent()`

**Variables:**
- camelCase for all: `reportData`, `analyticsResult`, `pdfFile`
- UPPERCASE for constants: `MAX_FILE_SIZE`, `SUPPORTED_FORMATS`
- Descriptive names, avoid abbreviations: `reportMetadata` not `rptMeta`

**Types (if TypeScript added):**
- PascalCase for all type definitions: `interface Report {}`, `type AnalyticsResult = {}`

## Where to Add New Code

**New Feature (e.g., new report type):**
- Primary code: `src/services/NewFeatureService.js`
- Models: `src/models/NewFeatureModel.js`
- Controllers: `src/controllers/NewFeatureController.js` (if HTTP)
- Tests: `tests/unit/services/NewFeatureService.test.js`
- Utilities: `src/utils/newFeatureHelpers.js` (if needed)

**New Component/Module:**
- Create a new directory under `src/services/` or `src/utils/`
- Export a single responsible module
- Use barrel exports in `src/[module]/index.js` if module contains multiple files
- Add tests in corresponding `tests/` structure

**Utilities:**
- Shared helpers: `src/utils/` directory
- File naming based on functionality: `validators.js`, `formatters.js`, `fileHelpers.js`
- Export all functions for use across codebase
- Include unit tests in `tests/unit/utils/`

**Constants and Configuration:**
- Hardcoded constants: `src/constants.js` or `src/config/constants.js`
- Environment-dependent config: Load from environment variables in `index.js`
- Magic numbers should be extracted to named constants

## Special Directories

**`.planning/codebase/`:**
- Purpose: Generated codebase analysis documents
- Generated: Yes (created by codebase analysis tools)
- Committed: Yes (safe to commit)
- Contents: ARCHITECTURE.md, STRUCTURE.md, STACK.md, etc.

**`.claude/`:**
- Purpose: Claude Code internal configuration
- Generated: Yes
- Committed: Depends on team preference (workspace-specific)

**`.git/`:**
- Purpose: Git version control history
- Generated: Yes
- Committed: No

---

*Structure analysis: 2026-02-18*
