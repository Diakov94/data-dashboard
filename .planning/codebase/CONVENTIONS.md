# Coding Conventions

**Analysis Date:** 2026-02-18

## Project Status

**Current State:** Initial scaffolding - no source code exists yet.

This document provides conventions to follow when source code is implemented. Entry point will be `index.js` per `package.json` configuration.

## Naming Patterns

**Files:**
- JavaScript modules: `camelCase.js` (when implemented)
- Entry point: `index.js` (established)
- Directories: `kebab-case` for multi-word directories

**Functions:**
- Standard functions: `camelCase()`
- Constructor/Class names: `PascalCase`
- Private methods: `_methodName()` prefix convention

**Variables:**
- Constants: `UPPER_SNAKE_CASE`
- Regular variables: `camelCase`
- Boolean variables: `isActive`, `hasData`, `canProcess` prefix

**Types:**
- If TypeScript is adopted: `PascalCase` for types and interfaces
- Example: `interface DashboardConfig`, `type ReportData`

## Code Style

**Formatting:**
- No linter currently configured (per CLAUDE.md)
- Recommend: ESLint when scaling
- Recommend: Prettier for auto-formatting (default: 2-space indentation)
- When implemented, align with JavaScript standard conventions (ES6+)

**Linting:**
- Will require configuration when source code is created
- Suggested: ESLint with recommended ruleset
- Consider: TypeScript support if type safety is needed

## Import Organization

**Order:**
1. Node.js core modules (`fs`, `path`, `http`)
2. Third-party dependencies (`npm` packages)
3. Local application modules (relative imports)
4. Index/barrel files last if used

**Path Aliases:**
- Not currently configured
- If needed for deep nested modules, configure in `package.json` or `jsconfig.json`

**Example pattern to follow:**
```javascript
// Core Node modules
const fs = require('fs');
const path = require('path');

// Third-party
const express = require('express');

// Local modules
const reportParser = require('./lib/reportParser');
const dashboardConfig = require('./config');
```

## Error Handling

**Patterns:**
- Use `try/catch` blocks for async operations
- Return error objects in callbacks when using callback-based APIs
- Propagate meaningful error messages (not generic "error occurred")
- Include context in error messages (e.g., file path, user action)

**Example:**
```javascript
try {
  const report = await parseReport(filePath);
} catch (error) {
  throw new Error(`Failed to parse report at ${filePath}: ${error.message}`);
}
```

## Logging

**Framework:** `console` (no external logger configured yet)

**Patterns:**
- Use `console.log()` for informational messages
- Use `console.error()` for error conditions
- Use `console.warn()` for warnings
- Include timestamp and context in messages when possible
- When scaling: migrate to structured logging (e.g., Winston, Pino)

**Example:**
```javascript
console.log(`[${new Date().toISOString()}] Loading report from ${filePath}`);
console.error(`[${new Date().toISOString()}] Failed to process report:`, error);
```

## Comments

**When to Comment:**
- Complex algorithmic sections (not obvious from code)
- Non-standard approaches or workarounds
- Critical business logic or calculations
- Configuration rationale
- Do NOT comment obvious code (e.g., `// increment counter`)

**JSDoc/TSDoc:**
- Not currently in use
- Recommended when TypeScript is adopted
- Use for public APIs and exported functions
- Format: `/** @param {type} name - description */`

**Example:**
```javascript
/**
 * Extracts metadata from PDF report
 * @param {Buffer} pdfBuffer - Raw PDF file contents
 * @returns {Object} Metadata including title, author, creation date
 * @throws {Error} If PDF is corrupted or unsupported
 */
function extractMetadata(pdfBuffer) {
  // implementation
}
```

## Function Design

**Size:**
- Keep functions to single responsibility
- Target: < 50 lines for readability
- Break down complex operations into smaller utilities

**Parameters:**
- Prefer simple parameters over complex objects when possible
- For many related parameters, use object destructuring
- Maximum 3-4 parameters before considering object parameter

**Return Values:**
- Always return consistently (don't mix returning value vs undefined)
- For functions that may fail, return object with `{success, data, error}`
- Explicitly return nothing (`return;`) or undefined if no meaningful value

**Example:**
```javascript
function processReport(pdfPath, { outputFormat = 'json', validate = true } = {}) {
  // Single responsibility
}

function analyzeData(data) {
  // Always returns: {success, data, error}
  if (!data) return { success: false, error: 'No data provided' };
  const result = analyze(data);
  return { success: true, data: result };
}
```

## Module Design

**Exports:**
- Use explicit named exports over default exports when possible
- Export a single function/class per file for single-responsibility modules
- Use `module.exports` for Node.js CommonJS (established in project)

**Barrel Files:**
- Consider `index.js` in directories for cleaner imports
- Example: `lib/index.js` to re-export utilities from `lib/`

**Example structure:**
```
lib/
├── index.js          // re-exports: reportParser, dashboardConfig
├── reportParser.js   // exports single function
└── dashboardConfig.js // exports config object
```

## Module Organization

**Application modules (when implemented):**
- `index.js` - Main entry point, bootstraps application
- `lib/` - Core business logic modules
- `config/` - Configuration files
- `utils/` - Shared utility functions
- `data/` - Data processing and transformation

---

*Convention analysis: 2026-02-18*
*Note: Established conventions based on Node.js/JavaScript best practices. To be followed as source code is implemented.*
