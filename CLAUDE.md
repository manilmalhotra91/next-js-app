# CLAUDE.md — AI Assistant Guide for next-js-app

## Project Overview

This is a Next.js application that demonstrates hybrid routing using both the **App Router** (modern, `app/` directory) and the **Pages Router** (legacy, `pages/` directory). It serves as a reference implementation with Jest testing, Docker deployment, and GitHub Actions CI/CD.

## Tech Stack

- **Framework:** Next.js (latest) with React 18
- **Language:** TypeScript (strict mode)
- **Testing:** Jest 29 + React Testing Library 15
- **Styling:** Global CSS (`styles/`) + CSS Modules (`*.module.css`)
- **Deployment:** Docker (Node 20-alpine) + Vercel-ready
- **CI/CD:** GitHub Actions (integration + deploy workflows)

## Project Structure

```
app/                    # App Router (Next.js 13+)
├── blog/[slug]/        # Dynamic route with generateMetadata
├── utils/              # Server-only utilities
├── layout.tsx          # Root layout (server component)
├── page.tsx            # Home page (server component)
├── counter.tsx         # Client component ("use client")
└── *.test.tsx          # Co-located tests

pages/                  # Pages Router (legacy)
├── home/index.tsx      # Pages Router home
└── _app.tsx            # Global app wrapper

__tests__/              # Centralized test directory
styles/                 # Global CSS
public/                 # Static assets (favicon, vercel.svg)
.github/workflows/      # CI/CD pipelines
```

## Commands

```bash
npm run dev          # Start development server
npm run build        # Production build
npm run start        # Start production server
npm test             # Run all tests (single run)
npm run test-watch   # Run tests in watch mode
npm run test:ci      # Run tests in CI mode
```

## Code Conventions

### Server vs. Client Components

- Components in `app/` are **server components by default** — no hooks or browser APIs allowed.
- Add `"use client"` directive at the top of files that need interactivity (hooks, event handlers, browser APIs).
- Use `import "server-only"` in utilities that must never run on the client (see `app/utils/add.ts`).

### TypeScript

- Strict mode is enabled — all code must pass strict type checking.
- Path aliases are configured in `tsconfig.json`:
  - `@/components/*` → `components/*`
  - `@/pages/*` → `pages/*`
  - `@/styles/*` → `styles/*`
- CSS Modules have type declarations in `types.d.ts`.

### Styling

- **Global CSS** is imported in `pages/_app.tsx` from `@/styles/global.css`.
- **CSS Modules** use the `*.module.css` convention (e.g., `pages/index.module.css`).
- No CSS-in-JS library is used.

### Metadata

- App Router pages use the `metadata` export or `generateMetadata` function for SEO.
- Pages Router pages use the `<Head>` component from `next/head`.

## Testing

### Setup

- Jest is configured via `jest.config.js` using the `next/jest` preset.
- `jest.setup.js` loads `@testing-library/jest-dom` for extended matchers (e.g., `toBeInTheDocument()`).
- Test environment is `jsdom` (configured in `jest.config.js`; some tests also use the `@jest-environment jsdom` pragma).

### Test Locations

Tests live in two places:
1. **Co-located** — next to source files in `app/` (e.g., `app/page.test.tsx`)
2. **Centralized** — in `__tests__/` directory (e.g., `__tests__/index.test.tsx`, `__tests__/snapshot.tsx`)

### Patterns

- Use `render()` and `screen` from `@testing-library/react` for component tests.
- Use `fireEvent` for simulating user interactions.
- Snapshot tests exist in `__tests__/snapshot.tsx` for regression detection.
- Pure utility functions are tested directly (see `app/utils/add.test.ts`).

### Running Tests

Always run `npm test` before committing to verify nothing is broken. The CI pipeline runs tests across Node.js 18.x, 20.x, and 22.x.

## CI/CD

### Integration (`integration.yml`)

- **Triggers:** Push to `main`, PRs targeting `main`
- **Jobs:** Build + Unit Tests across Node.js 18.x, 20.x, 22.x (matrix strategy)
- Uses npm caching for faster installs

### Deploy (`deploy.yml`)

- **Triggers:** Push to `main`
- Builds a Docker image (`harishneel/next-js-app`) and pushes to DockerHub
- Uses repository secrets for DockerHub authentication

## Docker

The `Dockerfile` uses `node:20-alpine`, installs dependencies, builds the app, exposes port 3000, and runs `npm start`.

## Key Files

| File | Purpose |
|---|---|
| `app/layout.tsx` | Root layout wrapping all App Router pages |
| `app/page.tsx` | App Router home page |
| `pages/_app.tsx` | Pages Router global wrapper (imports global CSS) |
| `jest.config.js` | Jest configuration with next/jest preset |
| `tsconfig.json` | TypeScript config (strict, path aliases) |
| `types.d.ts` | Type declarations for CSS Modules |
| `Dockerfile` | Production Docker build |
| `.github/workflows/` | CI/CD pipeline definitions |
