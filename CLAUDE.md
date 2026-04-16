# CLAUDE.md - AI Coding Agent Context

This file provides context for AI coding agents working on the Rowboat monorepo.

## Quick Reference Commands

```bash
# Electron App (apps/x)
cd apps/x && pnpm install          # Install dependencies
cd apps/x && npm run deps          # Build workspace packages (shared → core → preload)
cd apps/x && npm run dev           # Development mode (builds deps, runs app)
cd apps/x && npm run lint          # Lint check
cd apps/x/apps/main && npm run package   # Production build (.app)
cd apps/x/apps/main && npm run make      # Create DMG distributable

# Useful during development
cd apps/x && pnpm --filter @x/shared build   # Rebuild only shared package
cd apps/x && pnpm --filter @x/core build     # Rebuild only core package
```

## Monorepo Structure

```
rowboat/
├── apps/
│   ├── x/                 # Electron desktop app (focus of this doc)
│   ├── rowboat/           # Next.js web dashboard
│   ├── rowboatx/          # Next.js frontend
│   ├── cli/               # CLI tool
│   ├── python-sdk/        # Python SDK
│   └── docs/              # Documentation site
├── CLAUDE.md              # This file
└── README.md              # User-facing readme
```

## Electron App Architecture (`apps/x`)

The Electron app is a **nested pnpm workspace** with its own package management.

```
apps/x/
├── package.json           # Workspace root, dev scripts
├── pnpm-workspace.yaml    # Defines workspace packages
├── pnpm-lock.yaml         # Lockfile
├── apps/
│   ├── main/              # Electron main process
│   │   ├── src/           # Main process source
│   │   ├── forge.config.cjs   # Electron Forge config
│   │   └── bundle.mjs     # esbuild bundler
│   ├── renderer/          # React UI (Vite)
│   │   ├── src/           # React components
│   │   └── vite.config.ts
│   └── preload/           # Electron preload scripts
│       └── src/
└── packages/
    ├── shared/            # @x/shared - Types, utilities, validators
    └── core/              # @x/core - Business logic, AI, OAuth, MCP
```

### Build Order (Dependencies)

```
shared (no deps)
   ↓
core (depends on shared)
   ↓
preload (depends on shared)
   ↓
renderer (depends on shared)
main (depends on shared, core)
```

**The `npm run deps` command builds:** shared → core → preload

> **Note (personal):** If you see stale type errors after editing `@x/shared`, run
> `npm run deps` from `apps/x` again — the renderer's Vite HMR does not rebuild
> workspace packages automatically.

### Key Entry Points

| Component | Entry | Output |
|-----------|-------|--------|
| main | `apps/main/src/main.ts` | `.package/dist/main.cjs` |
| renderer | `apps/renderer/src/main.tsx` | `apps/renderer/dist/` |
| preload | `apps/preload/src/preload.ts` | `apps/preload/dist/preload.js` |

## Build System

- **Package manager:** pnpm (required for `workspace:*` protocol)
- **Main bundler:** esbuild (bundles to single CommonJS file)
- **Renderer bundler:** Vite
- **Packaging:** Electron Forge
- **TypeScript:** ES2022 target

### Why esbuild bundling?

pnpm uses symlinks for workspace packages. Electron Forge's dependency walker can't follow these symlinks. esbuild bundles everything into a single file, eliminating the need for node_modules in the packaged app.

## Key Files Reference

| Purpose | File |
|---------|------|
| Electron main entry | `apps/x/apps/main/src/m
