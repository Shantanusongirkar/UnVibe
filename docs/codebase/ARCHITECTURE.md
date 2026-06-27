# Architecture

## Core Sections (Required)

### 1) Architectural Style

- **Primary style:** Modular monolith with three independent services (frontend, backend, AI service), orchestrated via a monorepo.
- **Why this classification:**
  - The three apps (`web`, `api`, `ai-service`) are separate processes with distinct runtimes communicating over HTTP/WebSocket.
  - The `api` app is a single Express process containing tRPC endpoints, Prisma ORM, BullMQ job queue, and Socket.io server — NOT decomposed into microservices.
  - The `ai-service` is isolated as a separate Python process because it requires Python-specific ML/AI tooling.
- **Primary constraints:**
  1. All real-time features (Defend sessions, War Rooms) must go through the API's Socket.io server.
  2. AI operations (code generation, quiz generation, diff scoring) are only available through the Python AI service, which calls Claude.
  3. Database access is exclusively through the API's Prisma ORM — the web app never connects to PostgreSQL directly.

### 2) System Flow

```
Browser (Next.js App)
    │
    ├── HTTP/tRPC ──► Express API (PORT 4000) ──► PostgreSQL
    │                      │                        │
    │                      ├── BullMQ ──► Redis ─────┤ (job queue)
    │                      │                        │
    │                      └── Socket.io ────────────┘ (pub/sub for real-time)
    │                                 │
    ├── WebSocket (socket.io-client) ─┘
    │
    └── HTTP ──► Python FastAPI (PORT 8000) ──► Anthropic Claude API
                        │
                        └── Judge0 (planned, self-hosted sandboxed code execution)
```

**End-to-end flow (Decode → Rebuild → Defend):**

1. **User selects module** in browser → frontend calls backend API via tRPC
2. **Backend calls AI service** (`POST /generate/`) to get production code from Claude
3. **Code stored** in PostgreSQL via Prisma
4. **User annotates code** in Decode phase — saved via debounced API calls
5. **Quiz generated** by AI service (`POST /quiz/generate`) from annotations
6. **User writes solution** in Rebuild phase using Monaco editor
7. **On submit**, code sent to AI service (`POST /diff/`) for AST diff scoring
8. **Score stored** → IRS Engine recalculates user's Irreplaceability Score
9. **Defend session scheduled** via BullMQ — on fire, Claude generates questions from user's rebuild

### 3) Layer/Module Responsibilities

| Layer or module | Owns | Must not own | Evidence |
|-----------------|------|--------------|----------|
| `apps/web` (Frontend) | UI rendering, client state (Zustand), server state caching (React Query), routing, auth UI, animations | Direct database access, AI orchestration | `apps/web/src/app/layout.tsx`, `apps/web/src/app/providers.tsx` |
| `apps/api` (Backend) | HTTP/tRPC endpoints, authentication, business logic, database access (Prisma), job queue (BullMQ), real-time (Socket.io), logging, rate limiting | AI model calls, frontend rendering | `apps/api/src/index.ts` |
| `apps/ai-service` (AI) | LLM integration (Claude), code generation, quiz generation, diff scoring, defend Q&A, code execution (Judge0 — planned) | User data storage, authentication, frontend rendering | `apps/ai-service/app/main.py`, `apps/ai-service/app/routes/` |
| `packages/types` (Shared types) | TypeScript interfaces shared between web + api | Runtime logic, external dependencies | `packages/types/src/index.ts` |
| `infra/docker-compose.yml` (Infrastructure) | PostgreSQL, Redis for local development | Application code, migrations | `infra/docker-compose.yml` |

### 4) Reused Patterns

| Pattern | Where found | Why it exists |
|---------|-------------|---------------|
| tRPC (Type-safe RPC) | `apps/api/src/trpc.ts` — router, publicProcedure, middleware factories | Type-safe API contracts between frontend and backend |
| Prisma ORM (Repository pattern) | `apps/api/prisma/schema.prisma` + `apps/api/src/index.ts` singleton PrismaClient | Type-safe database access with migrations |
| BullMQ (Job queue) | `apps/api/src/index.ts` — `submissions` queue + worker | Async processing of Defend session scheduling |
| Socket.io (Pub/sub) | `apps/api/src/index.ts` — Socket.io server with wildcard CORS | Real-time features (Defend sessions, War Rooms) |
| NextAuth.js (Auth adapter) | `apps/web/src/auth.ts` — GitHub + Google OAuth providers; `apps/web/src/app/api/auth/[...nextauth]/route.ts` | Authentication with OAuth providers |
| Sentry (Error monitoring) | `apps/web/sentry.client.config.ts`, `sentry.server.config.ts`, `sentry.edge.config.ts`; `apps/api/src/index.ts` conditional Sentry init | Error tracking across frontend and backend |
| Turborepo (Monorepo orchestration) | `turbo.json` pipeline with dependency ordering | Parallel builds, consistent task execution across workspaces |

### 5) Known Architectural Risks

| Risk | Impact | Evidence |
|------|--------|----------|
| AI service routes are all **mocked** (no real Claude calls) | The core learning loop cannot function until real AI integration is implemented | `apps/ai-service/app/routes/generate.py` (commented-out Claude client), `routes/diff.py` (hardcoded response), `routes/quiz.py` (dummy questions), `routes/defend.py` (generic questions) |
| No service layer in any app — logic lives inline in routes | As the codebase grows, routes will become bloated, untestable, and hard to maintain | `apps/api/src/index.ts` has health check inline; no `services/` directory exists |
| API has no auth middleware on tRPC | Any endpoint added by default is public | `apps/api/src/trpc.ts` only exports `publicProcedure` — no `protectedProcedure` |
| CORS set to `allow_origins=["*"]` on both API and AI service | Insecure for production — allows any origin to make requests | `apps/api/src/index.ts` CORS config; `apps/ai-service/app/main.py` CORS config |

### 6) Evidence

- `apps/api/src/index.ts` — entry point showing Express, tRPC, BullMQ, Socket.io setup
- `apps/api/src/trpc.ts` — tRPC initialization (only publicProcedure exists)
- `apps/ai-service/app/main.py` — FastAPI app setup with mocked routes
- `apps/ai-service/app/routes/generate.py` — mocked Claude integration
- `apps/web/src/auth.ts` — NextAuth configuration
- `apps/web/src/app/providers.tsx` — client-side provider setup
- `turbo.json` — pipeline orchestration
