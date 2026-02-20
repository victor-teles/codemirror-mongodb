# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

`@codemirror/lang-sql` — SQL language support (syntax highlighting, autocompletion, multiple dialects) for the CodeMirror 6 editor. Version 6.10.0, MIT licensed.

## Commands

```bash
npm install              # Install dependencies
npm run prepare          # Build: cm-buildhelper src/sql.ts → dist/ (ESM + CJS + .d.ts)
npm test                 # Run tests: cm-runtests
```

There is no lint or format command configured. No tsconfig.json in root — build configuration is managed by `@codemirror/buildhelper`.

## Architecture

Three core source files in `src/`, each handling a distinct concern:

- **`sql.grammar`** — Lezer LR parser grammar defining the top-level `Script` → `Statement` structure. Uses `@external tokens` to delegate complex lexing to `tokens.ts`.

- **`tokens.ts`** — `ExternalTokenizer` implementation for SQL lexing. Handles string quoting styles, comments, numbers, operators, special variables, and dialect-specific syntax (PL/SQL quoting, double-dollar strings, bracket identifiers, etc.). Dialect behavior is controlled by a bitfield of flags (e.g., `backslashEscapes`, `hashComments`, `doubleDollarQuotedStrings`).

- **`sql.ts`** — Main entry point. Defines `SQLDialect` class (wraps parser + dialect config), the `sql()` function that creates a CodeMirror extension, and all predefined dialects: `StandardSQL`, `PostgreSQL`, `MySQL`, `MariaSQL`, `MSSQL`, `SQLite`, `Cassandra`, `PLSQL`. Each dialect is created via `SQLDialect.define()` with keywords, types, builtin functions, and tokenizer flags.

- **`complete.ts`** — Autocompletion engine. Provides `schemaCompletionSource` (hierarchical schema/table/column completion with alias resolution from FROM clauses) and `keywordCompletionSource` (dialect-aware keyword suggestions).

## Adding a New SQL Dialect

Call `SQLDialect.define()` with a `SQLDialectSpec` containing: `keywords`, `types`, `builtin` (space-separated strings), plus tokenizer flags from the dialect interface in `tokens.ts`. See existing dialect definitions at the bottom of `sql.ts` for examples.

## Tests

Tests are in `test/` using the `ist` assertion library:
- `test-complete.ts` — schema completion, alias resolution, quoted identifiers
- `test-tokens.ts` — tokenization across dialects (string literals, bit values, dollar-quoted strings)

## Code Style

- No semicolons
- ESM modules (`"type": "module"`)
- JSDoc comments on public API surfaces
