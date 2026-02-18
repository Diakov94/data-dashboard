# Testing Patterns

**Analysis Date:** 2026-02-18

## Project Status

**Current State:** No testing framework configured - placeholder `npm test` script exists.

Per CLAUDE.md: "No build, lint, or test tooling is configured yet."

This document provides testing patterns and structure to implement when source code is created.

## Test Framework

**Recommended Runner:**
- **Jest** [Version TBD] - Recommended for Node.js projects
  - Built-in assertion library (no additional dependency)
  - Excellent ES6+ support
  - Parallel test execution
  - Snapshot testing for UI/data validation

Alternative: **Vitest** - If using ESM modules or TypeScript

**Setup:** When implemented, create `jest.config.js` in project root:
```javascript
module.exports = {
  testEnvironment: 'node',
  testMatch: ['**/__tests__/**/*.js', '**/*.test.js', '**/*.spec.js'],
  collectCoverageFrom: ['lib/**/*.js', 'utils/**/*.js'],
};
```

**Assertion Library:**
- Jest's built-in assertions (`expect()`)
- No additional library needed

**Run Commands (to be configured):**
```bash
npm test                    # Run all tests
npm test -- --watch        # Watch mode
npm test -- --coverage     # Coverage report
npm test -- --testPathPattern=lib  # Run specific suite
```

## Test File Organization

**Location:**
- **Co-located pattern** (recommended): Test files adjacent to source files
- Alternative: `__tests__/` directory at module level

**Example layout (when source code exists):**
```
lib/
├── reportParser.js
├── reportParser.test.js      # Co-located
└── dashboardConfig.js
└── dashboardConfig.test.js

utils/
├── dataTransform.js
└── dataTransform.test.js
```

**Naming:**
- Pattern: `moduleName.test.js` or `moduleName.spec.js`
- Prefer `.test.js` for consistency
- Test files at same directory level as implementation

**Structure:**
```
lib/
  └── reportParser.js, reportParser.test.js
utils/
  └── helpers.js, helpers.test.js
__tests__/
  └── integration/  (for cross-module tests)
```

## Test Structure

**Suite Organization (recommended pattern):**

```javascript
const reportParser = require('../lib/reportParser');

describe('reportParser', () => {
  describe('extractMetadata', () => {
    test('should extract title from valid PDF', () => {
      const result = reportParser.extractMetadata(mockPdf);
      expect(result).toHaveProperty('title');
    });

    test('should throw on corrupted PDF', () => {
      expect(() => reportParser.extractMetadata(invalidPdf))
        .toThrow('Corrupted PDF');
    });
  });

  describe('parseContent', () => {
    test('should parse page content correctly', () => {
      // test implementation
    });
  });
});
```

**Patterns to follow:**

- **Setup:** Use `beforeEach()` for test data prep
```javascript
beforeEach(() => {
  // reset or initialize test data
  testData = loadFixture('sample-report.json');
});
```

- **Teardown:** Use `afterEach()` for cleanup
```javascript
afterEach(() => {
  // clean up temporary files
  fs.removeSync('./temp');
});
```

- **Assertion:** One logical assertion per test (can have multiple expect statements for related checks)
```javascript
test('should return complete report object', () => {
  const result = processReport(testData);
  expect(result).toHaveProperty('metadata');
  expect(result).toHaveProperty('content');
  expect(result).toHaveProperty('summary');
});
```

## Mocking

**Framework:** Jest's built-in mocking (`jest.mock()`, `jest.spyOn()`)

**Patterns:**

```javascript
// Mock entire module
jest.mock('../lib/pdfLibrary', () => ({
  parse: jest.fn().mockResolvedValue({ title: 'Test Report' })
}));

// Spy on existing module
const fs = require('fs');
jest.spyOn(fs, 'readFileSync')
  .mockReturnValue('mocked file content');

// Mock with different implementations per test
const mockParser = jest.fn()
  .mockReturnValueOnce({ success: true })
  .mockReturnValueOnce({ success: false });
```

**What to Mock:**
- External API calls (network requests)
- File system operations (use `mock-fs` for complex scenarios)
- Database queries
- Expensive operations (crypto, image processing)
- Time-dependent functions (mock `Date`, use `jest.useFakeTimers()`)

**What NOT to Mock:**
- Pure utility functions (helpers, calculations)
- Internal application logic (test actual behavior)
- Array/Object methods (test real interactions)
- Custom error classes (test real error handling)

**Example - File I/O mocking:**
```javascript
jest.mock('fs', () => ({
  readFileSync: jest.fn(() => 'test content'),
  writeFileSync: jest.fn(),
}));

test('should read report file', () => {
  const result = loadReport('report.pdf');
  expect(fs.readFileSync).toHaveBeenCalledWith('report.pdf');
});
```

## Fixtures and Factories

**Test Data Pattern (when implemented):**

Create `__tests__/fixtures/` directory:
```javascript
// __tests__/fixtures/reports.js
module.exports = {
  validReport: {
    metadata: { title: 'Q1 Report', pages: 10 },
    content: ['page 1 content', 'page 2 content'],
  },

  invalidReport: {
    // missing required fields
  },
};

// Usage in test file
const { validReport } = require('../__tests__/fixtures/reports');

test('should process valid report', () => {
  const result = processReport(validReport);
  expect(result.success).toBe(true);
});
```

**Location:**
- `__tests__/fixtures/` - Data files for tests
- `__tests__/factories/` - Factory functions for creating test objects (if needed)

**Example factory pattern:**
```javascript
// __tests__/factories/reportFactory.js
function createReport(overrides = {}) {
  return {
    id: Math.random().toString(36).substr(2, 9),
    title: 'Test Report',
    pages: 5,
    createdAt: new Date().toISOString(),
    ...overrides,
  };
}

module.exports = { createReport };
```

## Coverage

**Requirements:** Not enforced currently

**Recommended targets (when coverage is enabled):**
- Statements: 80%+
- Branches: 75%+
- Functions: 80%+
- Lines: 80%+

**View Coverage:**
```bash
npm test -- --coverage
# Generates coverage/ directory with HTML report
open coverage/lcov-report/index.html
```

**Coverage configuration (jest.config.js):**
```javascript
module.exports = {
  collectCoverageFrom: [
    'lib/**/*.js',
    'utils/**/*.js',
    '!lib/**/*.test.js',  // exclude test files
  ],
  coverageThreshold: {
    global: {
      branches: 75,
      functions: 80,
      lines: 80,
      statements: 80,
    },
  },
};
```

## Test Types

**Unit Tests:**
- Scope: Individual functions and modules
- Approach: Isolated testing with mocked dependencies
- Location: `lib/*.test.js`, `utils/*.test.js`
- Speed: Very fast (< 1 second per test)

**Example:**
```javascript
test('should validate report format', () => {
  const result = validateReport({ title: 'Test' });
  expect(result.isValid).toBe(true);
});
```

**Integration Tests:**
- Scope: Multiple modules working together
- Approach: Real file I/O, but mocked external APIs
- Location: `__tests__/integration/`
- Speed: Moderate (< 5 seconds per test)

**Example:**
```javascript
describe('Report Processing Pipeline', () => {
  test('should load, parse, and validate report', () => {
    const pdf = fs.readFileSync('./samples/report.pdf');
    const metadata = extractMetadata(pdf);
    const content = parseContent(pdf);
    expect(metadata).toBeDefined();
    expect(content).toBeDefined();
  });
});
```

**E2E Tests:**
- Status: Not currently applicable (no UI/server endpoints yet)
- Framework: Consider Playwright or Puppeteer if web dashboard is implemented
- When needed: Create `__tests__/e2e/` directory

## Common Patterns

**Async Testing:**

```javascript
// Using async/await
test('should load report asynchronously', async () => {
  const report = await loadReportAsync('./data/report.pdf');
  expect(report.title).toBe('Expected Title');
});

// Using .resolves and .rejects
test('should handle promise rejection', async () => {
  await expect(loadReportAsync('./invalid')).rejects.toThrow('Not found');
});
```

**Error Testing:**

```javascript
test('should throw on invalid input', () => {
  expect(() => processReport(null)).toThrow('Invalid report');
});

test('should include error context', () => {
  try {
    riskyOperation();
  } catch (error) {
    expect(error.message).toContain('specific context');
  }
});

// Async error testing
test('should reject with meaningful error', async () => {
  await expect(asyncRiskyOp()).rejects
    .toEqual(expect.objectContaining({
      message: expect.stringContaining('failed'),
      code: 'INVALID_STATE',
    }));
});
```

**Snapshot Testing (for data structures):**

```javascript
test('should generate report summary with expected structure', () => {
  const summary = generateSummary(testReport);
  expect(summary).toMatchSnapshot();
  // First run creates snapshot, subsequent runs compare
});
```

## Pre-commit Testing (Future)

When scaling, configure Husky + lint-staged:
```bash
npm install husky lint-staged --save-dev
```

Configuration in `package.json`:
```json
{
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged"
    }
  },
  "lint-staged": {
    "*.js": ["npm test --", "--testPathPattern=<FILE>"]
  }
}
```

---

*Testing analysis: 2026-02-18*
*Note: No testing framework currently implemented. Recommendations based on Node.js best practices. To be implemented when source code is created.*
