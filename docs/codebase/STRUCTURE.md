# Codebase Structure

## Core Sections (Required)

### 1) Top-Level Map

| Path | Purpose | Evidence |
|------|---------|----------|
| `apps/` | Contains three application sub-projects: web (frontend), api (backend), ai-service (AI) | `ls apps/`, `pnpm-workspace.yaml` |
| `apps/web/` | Next.js 14 frontend with App Router | `apps/web/package.json` |
| `apps/api/` | Express + tRPC + Prisma backend API | `apps/api/package.json`, `apps/api/src/index.ts` |
| `apps/ai-service/` | Python FastAPI service for all AI logic | `apps/ai-service/requirements.txt`, `apps/ai-service/app/main.py` |
| `packages/` | Shared packages within the monorepo | `pnpm-workspace.yaml` |
| `packages/types/` | Shared TypeScript interfaces and types (`@unvibe/types`) | `packages/types/src/index.ts`, `packages/types/package.json` |
| `infra/` | Docker Compose files for local development infrastructure | `infra/docker-compose.yml` |
| `.github/` | GitHub workflows and CI/CD configuration | `.github/workflows/discord.yml` |
| `.env.example` | Template for required environment variables | `.env.example` |
| `turbo.json` | Turborepo pipeline configuration | `turbo.json` |
| `pnpm-workspace.yaml` | pnpm workspace definitions (`apps/*`, `packages/*`) | `pnpm-workspace.yaml` |
| `eslint.base.json` | Base ESLint configuration shared across workspaces | `eslint.base.json` |
| `tsconfig.base.json` | Base TypeScript configuration shared across workspaces | `tsconfig.base.json` |

### 2) Entry Points

| App | Entry Point | Port | How Started |
|-----|-------------|------|-------------|
| Frontend (web) | `apps/web/src/app/layout.tsx` (root layout) | 3000 | `pnpm --filter web dev` → Next.js dev server |
| Backend (api) | `apps/api/src/index.ts` | 4000 | `pnpm --filter api dev` → tsx watch |
| AI Service | `apps/ai-service/app/main.py` | 8000 | `pnpm --filter ai-service` → uvicorn |

### 3) Module Boundaries

| Boundary | What belongs here | What must not be here |
|----------|-------------------|------------------------|
| `apps/web/src/app/` | Page components, layouts, API routes (NextAuth) | Business logic, DB access, AI service calls (should go through API) |
| `apps/web/src/lib/` | Utilities, auth config, state stores, tRPC client | Page-level components, routing |
| `apps/web/src/components/` | [TODO] Reusable UI components (shadcn). Not yet created. | Backend logic, API routes |
| `apps/api/src/routers/` | [TODO] tRPC route handlers. Not yet created. | Data access (should be in services), AI orchestration |
| `apps/api/src/services/` | [TODO] Business logic services. Not yet created. | HTTP handling, route definitions |
| `apps/api/prisma/` | Database schema and migrations | Business logic, API handlers |
| `apps/api/src/middleware/` | [TODO] Express middleware. Not yet created. | Route definitions, business logic |
| `apps/ai-service/app/routes/` | FastAPI endpoint definitions | Business logic (should be in services), data access |
| `apps/ai-service/app/services/` | [TODO] AI service wrappers. Not yet created. | Route definitions, HTTP concerns |
| `apps/ai-service/app/prompts/` | [TODO] Versioned Claude prompt templates. Not yet created. | Application logic, HTTP handling |
| `packages/types/src/` | Shared TypeScript interfaces used by web + api | Implementation code, runtime dependencies |

### 4) Naming and Organization Rules

- File naming pattern:
  - **TypeScript/React (web + api):** kebab-case for files (`route.ts`, `providers.tsx`, `layout.tsx`), PascalCase for page/component files (`page.tsx` as convention)
  - **Python (ai-service):** snake_case for files (`generate.py`, `quiz.py`)
  - **Config files:** kebab-case (`docker-compose.yml`, `eslint.base.json`, `tsconfig.base.json`)
- Directory organization pattern: **Layer-by-layer** within each app (e.g., `routes/`, `services/`, `db/` in planned structure; `app/`, `lib/` in web)
- Import aliasing or path conventions:
  - Web (apps/web): `@/*` maps to `src/*` (e.g., `@/app/providers`, `@/lib/utils`)
  - API (apps/api): No path aliases — uses relative imports
  - AI Service: Standard Python imports
  - Shared: Workspace package names via pnpm (`@unvibe/types`)

### 5) Evidence

- `apps/web/tsconfig.json` (path alias `@/*`)
- `apps/api/tsconfig.json` (no path aliases, extends `tsconfig.base.json`)
- `pnpm-workspace.yaml`
- `turbo.json`
- File tree via `Get-ChildItem -Recurse -Depth 3`
