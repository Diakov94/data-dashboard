# Technology Stack

**Analysis Date:** 2026-02-18

## Languages

**Primary:**
- JavaScript - Used for application code. Entry point at `index.js` per `package.json` main field.

**Secondary:**
- None currently active

## Runtime

**Environment:**
- Node.js v22.22.0 (installed at analysis time)
- No `.nvmrc` file present; runtime version not pinned

**Package Manager:**
- npm v10.9.4 (installed at analysis time)
- `package.json` present, no lockfile (`package-lock.json`) present in repository

## Frameworks

**Core:**
- None currently installed - Project is in initial scaffolding stage

**Testing:**
- None configured - `npm test` script is a placeholder that exits with error (per `package.json` line 7)

**Build/Dev:**
- None configured - No build tools, linters, or development utilities installed

## Key Dependencies

**Critical:**
- None - No production dependencies specified in `package.json`

**Infrastructure:**
- None - No infrastructure-as-code dependencies

## Configuration

**Environment:**
- No `.env` file or environment configuration files present
- MCP servers configured in `.mcp.json` (project-level configuration):
  - `context7` — Requires `CONTEXT7_API_KEY` environment variable
  - `sequential-thinking` — No credentials required

**Build:**
- No build configuration files present
- No `tsconfig.json`, webpack, rollup, or other build tools configured

## Platform Requirements

**Development:**
- Node.js runtime (minimum version not specified; tested with v22.22.0)
- npm or equivalent package manager
- No OS-specific requirements detected

**Production:**
- Node.js runtime required (version not specified)
- No containerization (Docker, Podman) configured
- No deployment infrastructure (Terraform, CloudFormation) present

---

*Stack analysis: 2026-02-18*
