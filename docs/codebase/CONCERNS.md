# Codebase Concerns

## Core Sections (Required)

### 1) Top Risks (Prioritized)

| Severity | Concern | Evidence | Impact | Suggested action |
|----------|---------|----------|--------|------------------|
| **Critical** | All AI service endpoints return hardcoded mock data — no real Claude integration | `apps/ai-service/app/routes/generate.py` (commented-out Anthropic client, returns mock), `routes/diff.py` (hardcoded diff), `routes/quiz.py` (dummy questions), `routes/defend.py` (generic questions) | The core learning loop (Decode → Rebuild → Defend) cannot function; the product has no working AI features | Wire up real Anthropic Claude API calls; implement proper diff engine, quiz generation, and defend Q&A |
| **High** | No test coverage anywhere in the monorepo | No test files found; no test scripts in any `package.json`; `turbo.json` has a `test` pipeline but no underlying script | Every change risks regressions; no safety net for refactoring; cannot ship with confidence | Select a test framework (Jest/Vitest for TS, pytest for Python); add initial test suite |
| **High** | No auth protection on tRPC endpoints — all procedures are `publicProcedure` | `apps/api/src/trpc.ts` only exports `publicProcedure`; no `protectedProcedure` or auth middleware for tRPC | Any added tRPC endpoint is public by default; users could access others' data | Implement auth middleware for tRPC (wrap NextAuth session check); create `protectedProcedure` |
| **High** | CORS allows any origin (`*`) on both API and AI Service | `apps/api/src/index.ts` CORS config: no explicit origin restriction; `apps/ai-service/app/main.py`: `allow_origins=["*"]` | In production, any website can make requests to the API; CSRF-style attacks possible | Restrict CORS to specific frontend origins in production |
| **High** | No service layer exists — logic is written inline in entry points and routes | `apps/api/src/index.ts` has health check inline; `apps/ai-service/app/routes/` files contain all logic inline; no `services/` directory exists in any app | Routes will become bloated and untestable; business logic cannot be reused | Extract business logic into service modules; keep routes thin |

### 2) Technical Debt

| Debt item | Why it exists | Where | Risk if ignored | Suggested fix |
|-----------|---------------|-------|-----------------|---------------|
| Mocked AI responses | AI service was scaffolded before Claude integration was implemented | All 4 routes in `apps/ai-service/app/routes/` | Core product functionality does not exist; demo-only state | Implement real Claude API calls, diff engine, quiz generation |
| No .env file present | Setup not completed — developer must copy from `.env.example` | Root of repo | First-time setup fails silently; apps may crash looking for env vars | Add setup validation that checks required env vars on startup |
| BullMQ queue + worker created with no actual job processing | Scaffolded but no real jobs dispatched or processed | `apps/api/src/index.ts` (`submissions` queue/worker) | Queue infrastructure exists but provides no value; confusing to on-boarders | Either implement real job dispatch or remove unused queue scaffolding |
| Socket.io server with no rooms or events | Scaffolded but no real-time features implemented | `apps/api/src/index.ts` (Socket.io init with connect/disconnect logging only) | WebSocket infrastructure exists but is unused | Implement Defend session rooms or remove until needed |
| Prisma migrations directory | [TODO] — no `prisma/migrations/` directory found | `apps/api/prisma/` | Cannot create database tables; app will fail at startup | Run `npx prisma migrate dev` to generate initial migration |
| No Prettier config | Not added during project setup | Monorepo root | Inconsistent code formatting across contributions | Add `.prettierrc` with team-agreed defaults |
| `packages/config/` deleted, configs moved to root without cleanup | Commit `be054b1` ("cleanup") deleted the package | Root config files (`eslint.base.json`, `tsconfig.base.json`) | Workspace references to `@unvibe/config` may break if not updated | Ensure no remaining imports from `@unvibe/config` |

### 3) Security Concerns

| Risk | OWASP category | Evidence | Current mitigation | Gap |
|------|----------------|----------|--------------------|-----|
| Public tRPC procedures (no auth) | A01 (Broken Access Control) | `apps/api/src/trpc.ts` — only `publicProcedure` exported | None | Implement `protectedProcedure` with session validation |
| CORS wildcard in production | A05 (Security Misconfiguration) | `apps/api/src/index.ts`: no explicit origin; `apps/ai-service/app/main.py`: `allow_origins=["*"]` | None | Restrict to specific origins per environment |
| No input validation on API (beyond tRPC/Zod) | A03 (Injection) | [TODO] — no route handlers exist yet to audit | Zod is available but not yet wired to all procedures | Ensure all mutation procedures validate input with Zod |
| NextAuth session secret uses default value | A02 (Cryptographic Failures) | `.env.example` shows placeholder `"some-very-secure-random-secret-key-at-least-32-chars-long"` | None — must be changed in production | Document requirement to generate unique `NEXTAUTH_SECRET` |
| Redis exposed without auth | A05 (Security Misconfiguration) | `infra/docker-compose.yml`: Redis started without password | None (local dev only) | Add Redis password for production; use AUTH command |
| Sensitive data in logs | A09 (Security Logging and Monitoring Failures) | No redaction patterns established anywhere | None | Implement PII/secret redaction in pino logger |

### 4) Performance and Scaling Concerns

| Concern | Evidence | Current symptom | Scaling risk | Suggested improvement |
|---------|----------|-----------------|-------------|-----------------------|
| No database connection pooling | `apps/api/src/index.ts` — single `PrismaClient` singleton | N/A (no production traffic) | Connection exhaustion under load | Use Prisma's connection pool config or PgBouncer |
| AI service endpoints are synchronous | `apps/ai-service/app/routes/generate.py` — no async handling pattern | N/A (all mocked) | Slow AI calls block the request thread | Use FastAPI async endpoints; offload long generations to background tasks |
| No caching layer configured | Redis is available but not used for caching | N/A | Repeated AI calls or DB queries on same data | Implement response caching with Redis for AI results |
| No rate limiting | `apps/api/src/middleware/` doesn't exist yet | N/A | Abuse of API endpoints possible | Implement rate limiting on all public endpoints |

### 5) Fragile/High-Churn Areas

| Area | Why fragile | Churn signal | Safe change strategy |
|------|-------------|-------------|----------------------|
| `apps/ai-service/app/routes/` | All 4 route files are mocked stubs that will be completely rewritten when real Claude integration is implemented | High churn in git history — these are the most recently modified files | Write integration tests first, then replace implementations one endpoint at a time |
| `apps/api/prisma/schema.prisma` | Schema changes cascade to all dependent code (types, services, routers) | Changed multiple times in early commits before reaching current state | Add migration tests; use Prisma's `db push` for rapid iteration during development |
| `apps/api/src/index.ts` | Monolithic entry point — Express, tRPC, BullMQ, Socket.io, Sentry, CORS all configured in one file | Modified in most API-related commits | Split into separate config modules (e.g., `config/express.ts`, `config/socket.ts`) |
| `apps/web/src/auth.ts` | Auth provider configuration that bridges frontend and backend (NextAuth) | Modified across multiple scaffolding commits | Keep auth config isolated; test with integration tests |
| Shared types (`packages/types/src/index.ts`) | Changes affect both web and api consumers | Modified as schema evolved | Version the types package; use breaking-change protocol |

### 6) `[ASK USER]` Questions

1. **[ASK USER]** What test framework would you like to use for the TypeScript apps (web + api)? Options: Vitest (modern, fast), Jest (widely adopted), or Playwright (E2E focused).
2. **[ASK USER]** What test framework would you like for the Python AI service? Options: pytest (standard), pytest-asyncio (for async endpoints).
3. **[ASK USER]** What production deployment targets are planned? Vercel (web) + Railway/Render (api) as stated in README, or have you selected other providers?
4. **[ASK USER]** Is a no-SQL database or additional data store expected beyond PostgreSQL + Redis? The current schema covers all models relationally.
5. **[ASK USER]** Should we implement a `protectedProcedure` wrapper for tRPC that checks NextAuth session, or use a different auth strategy for API endpoints?
6. **[ASK USER]** What formatter preference do you want? Options: Prettier with default config, or a more customized setup with specific print width, semi-colons, trailing commas, etc.?
7. **[ASK USER]** The current roadmap mentions Judge0 for sandboxed code execution — is this still planned, or should we deprioritize it?

### 7) Evidence

- `apps/ai-service/app/routes/generate.py` — mocked Claude integration
- `apps/ai-service/app/routes/diff.py` — hardcoded diff response
- `apps/ai-service/app/routes/quiz.py` — dummy quiz generation
- `apps/ai-service/app/routes/defend.py` — generic defend questions
- `apps/api/src/trpc.ts` — only publicProcedure
- `apps/api/src/index.ts` — monolithic entry point
- `apps/api/prisma/schema.prisma` — Prisma schema
- `apps/web/src/auth.ts` — NextAuth config
- `infra/docker-compose.yml` — PostgreSQL + Redis config
- `.env.example` — required env vars
- `turbo.json` — test pipeline without test script
- Git log (recent 20 commits showing churn patterns)
