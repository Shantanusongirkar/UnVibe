# External Integrations

## Core Sections (Required)

### 1) Integration Inventory

| System | Type | Purpose | Auth model | Criticality | Evidence |
|--------|------|---------|------------|-------------|----------|
| PostgreSQL 16 | Database (relational) | Primary data store for all application data | Password via `DATABASE_URL` connection string | Critical | `infra/docker-compose.yml`, `apps/api/prisma/schema.prisma` |
| Redis 7 | Cache + Queue + Pub/Sub | BullMQ job queue, Socket.io pub/sub, general caching | None (local, no password configured) | Critical | `infra/docker-compose.yml`, `apps/api/src/index.ts` |
| Anthropic Claude API | External API (LLM) | Code generation, quiz generation, defend Q&A | API key (`ANTHROPIC_API_KEY`) | Critical | `.env.example`, `apps/ai-service/requirements.txt` (NOT YET IMPLEMENTED) |
| GitHub OAuth | External API (Auth) | OAuth sign-in for users | Client ID + Secret (`GITHUB_CLIENT_ID`, `GITHUB_CLIENT_SECRET`) | High | `apps/web/src/auth.ts` |
| Google OAuth | External API (Auth) | OAuth sign-in for users | Client ID + Secret (`GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`) | High | `apps/web/src/auth.ts` |
| Cloudflare R2 | Object Storage | Store code snapshots, PDF reports, assets | Account ID + Access Key + Secret Key (`R2_*` env vars) | Medium | `.env.example` |
| Resend | External API (Email) | Transactional emails (digest, notifications) | API key (`RESEND_API_KEY`) | Medium | `.env.example` |
| Sentry | Monitoring | Error tracking across web + api | DSN via `SENTRY_DSN_WEB`, `SENTRY_DSN_API` env vars | Low (local dev) | `apps/web/sentry.client.config.ts`, `apps/api/src/index.ts` |
| PostHog | Analytics | Product analytics | Public key via `NEXT_PUBLIC_POSTHOG_KEY` | Low (local dev) | `.env.example` |
| Judge0 | Self-hosted service | Sandboxed code execution (planned) | [TODO] — self-hosted, not yet deployed | Low (not implemented) | `README.md` tech stack mentions |

### 2) Data Stores

| Store | Role | Access layer | Key risk | Evidence |
|-------|------|--------------|----------|----------|
| PostgreSQL 16 | Primary database — all users, tracks, modules, submissions, defend sessions, IRS scores, war rooms | Prisma ORM via `apps/api` | No connection pooling configured; single `PrismaClient` singleton | `apps/api/prisma/schema.prisma`, `apps/api/src/index.ts` |
| Redis 7 | Job queue (BullMQ), real-time pub/sub (Socket.io), caching | `ioredis` via BullMQ + Socket.io adapter | No auth or TLS configured for local dev; no persistence policy set for cache use | `infra/docker-compose.yml`, `apps/api/src/index.ts` |
| Cloudflare R2 | Object storage for code snapshots, PDF reports | [TODO] — SDK not yet imported | [TODO] — not yet implemented | `.env.example` (env vars defined) |

### 3) Secrets and Credentials Handling

- Credential sources: All secrets and credentials are read from environment variables (`.env` file at repo root).
- Hardcoding checks: No hardcoded secrets found. All API keys, OAuth secrets, and database passwords reference env vars.
- Rotation or lifecycle notes: **Unknown.** No secrets manager configured. No credential rotation mechanism in place.

### 4) Reliability and Failure Behavior

- Retry/backoff behavior:
  - **BullMQ:** Built-in retry mechanism for job queue (configured via queue options — [TODO] exact config not visible)
  - **Anthropic Claude:** Not yet implemented, so no retry behavior exists
  - **Database:** Prisma has no explicit retry configured in the codebase
- Timeout policy:
  - **Express:** No global timeout configured
  - **AI Service:** No timeout configured on any endpoint
  - **Socket.io:** Default timeout (no custom configuration found)
- Circuit-breaker or fallback behavior: **None found.** No circuit breaker (e.g., opossum) or fallback patterns implemented.

### 5) Observability for Integrations

- Logging around external calls:
  - **API:** pino logger initialized but no integration-level logging found
  - **AI Service:** loguru dependency declared but no logging calls in route files
  - **Sentry:** Conditional Sentry initialization — request/error handlers for Express
- Metrics/tracing coverage: **None found.** No metrics collection (Prometheus, OpenTelemetry) or distributed tracing.
- Missing visibility gaps: **All of them.** No instrumentation on database queries, Redis/BullMQ operations, external API calls, or WebSocket events.

### 6) Evidence

- `apps/web/src/auth.ts` — GitHub + Google OAuth provider config
- `apps/api/src/index.ts` — PrismaClient, BullMQ, Socket.io, Sentry setup
- `apps/ai-service/app/routes/generate.py` — commented-out Anthropic client
- `apps/ai-service/requirements.txt` — Anthropic SDK dependency
- `.env.example` — all expected environment variables
- `infra/docker-compose.yml` — PostgreSQL and Redis configuration
