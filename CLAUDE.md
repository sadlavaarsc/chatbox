# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Chatbox is a desktop AI chat client supporting multiple LLM providers (OpenAI, Claude, Gemini, Ollama, etc.), built with Electron + React + TypeScript. Available on Windows, Mac, Linux, Web, iOS, and Android.

## Build & Development

**Prerequisites:**
- Node.js v20.x – v22.x
- pnpm >= 10.0.0
- **Conda environment:** `chatbox-env` — install dependencies in this environment only, not in `base` or other environments

**Important:** Use `pnpm`, not `npm`. Using `npm` may cause build and debug failures.

**Core Commands:**
```bash
pnpm install              # Install dependencies
pnpm run dev              # Start development mode (Electron)
pnpm run dev:web          # Start development mode (Web only)
pnpm run build            # Build for production
pnpm run package          # Package installer for current platform
pnpm run package:all      # Package installers for all platforms
pnpm test                 # Run tests with Vitest
pnpm run test:ui          # Run tests with UI
pnpm run lint             # Lint with Biome
pnpm run lint:fix         # Auto-fix lint issues
pnpm run format           # Format with Biome
pnpm run check            # TypeScript type check
```

**Mobile Development:**
```bash
pnpm run mobile:sync:ios    # Build and sync iOS
pnpm run mobile:sync:android # Build and sync Android
pnpm run mobile:ios         # Open in Xcode
pnpm run mobile:android     # Open in Android Studio
```

## Architecture

**Three-process Electron structure:**
- `src/main/` - Main process (Node.js, Electron APIs, IPC handlers)
- `src/preload/` - Preload script (contextBridge for renderer-main communication)
- `src/renderer/` - Renderer process (React UI, runs in Chromium)
- `src/shared/` - Shared types, utilities, and provider definitions

**Key renderer directories:**
- `routes/` - TanStack Router file-based routes
- `stores/` - Jotai atoms and store management (settings, sessions, UI state)
- `components/` - React components organized by feature
- `hooks/` - Custom React hooks
- `i18n/` - Internationalization (12+ languages)
- `setup/` - App initialization (Sentry, migrations, MCP bootstrap)

**Provider system:**
`src/shared/providers/` defines the AI provider registry with per-provider configurations and model definitions. Adding a new provider involves:
1. Create definition in `src/shared/providers/definitions/`
2. Register in `src/shared/providers/registry.ts`
3. Add model configuration in corresponding `models/` subdirectory

**Knowledge Base:**
`src/main/knowledge-base/` handles RAG functionality including file parsing, embeddings, vector storage (LibSQL), and document management.

**State management:**
- Jotai for React state (atoms in `src/renderer/stores/atoms/`)
- TanStack Query for server state (`src/renderer/stores/queryClient.ts`)
- Electron store (`src/main/store-node.ts`) for persistent settings

**Build system:**
- electron-vite for multi-process bundling
- Vite plugins: TanStack Router, React, Sentry
- Manual chunks configured for vendor splitting (AI SDK, Mantine, Mermaid)
- Path aliases: `@/` → `src/renderer/`, `@shared/` → `src/shared/`

## Testing

Test files are in `test/integration/` and alongside source files (`*.test.ts`):
```bash
pnpm test                              # Run all tests
pnpm run test:integration              # Integration tests only
pnpm run test:file-conversation        # File parsing tests
pnpm run test:model-provider           # Provider tests
```

## Code Style

**Linting:** Biome (not ESLint)
- `biome.json` config: 120 char line width, single quotes, semicolons as needed
- Formatter and linter both use Biome
- `lint-staged` runs on pre-commit for `*.{js,jsx,ts,tsx}`

**Key conventions:**
- Path aliases: `@/` for renderer, `@shared/` for shared code
- IPC naming: `ipcMain.handle()` in main, `window.electronAPI.invoke()` in renderer
- Store keys: camelCase, defined in `src/shared/types/settings.ts`
- Component files: PascalCase, colocated styles and tests
- Utilities: snake_case file names in `utils/` directories

## Important Files

- `electron.vite.config.ts` - Build configuration, Sentry setup, chunk splitting
- `src/main/main.ts` - Main process entry, window management, IPC handlers
- `src/renderer/index.tsx` - Renderer entry, migrations, app initialization
- `src/preload/index.ts` - contextBridge exposing electronAPI to renderer
- `src/shared/types.ts` - Core TypeScript types
- `src/shared/electron-types.ts` - IPC type definitions
- `biome.json` - Linting and formatting rules
- `package.json` - Scripts and dependency versions
