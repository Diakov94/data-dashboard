# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Interactive analytics dashboard for PDF reports. Entry point is `index.js`.

## Commands

No build, lint, or test tooling is configured yet. The `npm test` script is a placeholder.

## MCP Servers

Two MCP servers are configured in `.mcp.json` (project-level, safe to commit):
- **context7** — library documentation lookup. Requires `CONTEXT7_API_KEY` env var (`export CONTEXT7_API_KEY=...`).
- **sequential-thinking** — structured reasoning tool. No credentials required.

## Architecture

The project is in its initial scaffolding stage — no source code exists yet. When implementing, `index.js` should serve as the application entry point per `package.json`.
