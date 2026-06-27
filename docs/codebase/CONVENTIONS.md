# Coding Conventions

## Core Sections (Required)

### 1) Naming Rules

| Item | Rule | Example | Evidence |
|------|------|---------|----------|
| Files (TypeScript) | kebab-case with lowercase | `providers.tsx`, `layout.tsx`, `globals.css`, `route.ts` | `apps/web/src/app/` directory listing |
| Files (Python) | snake_case | `generate.py`, `quiz.py`, `defend.py`, `diff.py` | `apps/ai-service/app/routes/` directory listing |
| Files (Config) | kebab-case | `docker-compose.yml`, `eslint.base.json`, `tsconfig.base.json` | Root directory listing |
| Page components (Next.js) | Lowercase, `page.tsx` convention | `page.tsx` for homepage | `apps/web/src/app/page.tsx` |
| Layout components (Next.js) | Lowercase, `layout.tsx` convention | `layout.tsx` for root layout | `apps/web/src/app/layout.tsx` |
| Functions (TypeScript) | camelCase | `export function cn(...inputs)` | `apps/web/src/lib/utils.ts` |
| Functions (Python) | snake_case | [TODO] — no service functions yet | No implementation exists |
| Types/interfaces (TypeScript) | PascalCase | `User`, `Track`, `Module`, `Submission`, `DefendSession`, `WarRoom`, `IRSScore` | `packages/types/src/index.ts` |
| Constants/env vars | UPPER_SNAKE_CASE | `DATABASE_URL`, `REDIS_URL`, `ANTHROPIC_API_KEY` | `.env.example` |
| Branch naming | `feat/`, `fix/`, `chore/`, `docs/` prefix | `feat/your-feature-name` | `CONTRIBUTING.md` |

### 2) Formatting and Linting

- Formatter: **No formatter configured.** No `.prettierrc` or equivalent exists in the monorepo.
- Linter: **ESLint ^8** with config:
  - Root: `eslint.base.json` — `eslint:recommended`, `es2022`, `no-unused-vars: warn`, `no-console: off`
  - Web: `apps/web/.eslintrc.json` — extends `next/core-web-vitals`, `next/typescript`
  - API: `[TODO]` — no ESLint config found in apps/api/; inherits from base
- Most relevant enforced rules: `no-unused-vars: warn`, `no-console: off` (base)
- Run commands: `pnpm lint` (runs turbo lint across all workspaces)

### 3) Import and Module Conventions

- Import grouping/order: **No explicit convention enforced.** No ESLint import ordering plugin configured.
- Alias vs relative import policy:
  - **Web (apps/web):** Uses `@/` alias for all imports (e.g., `@/lib/utils`, `@/app/providers`). Configured in `tsconfig.json` paths.
  - **API (apps/api):** Uses relative imports (no path aliases configured).
  - **AI Service:** Standard Python imports.
  - **Packages:** Uses `@unvibe/types` workspace package name.
- Public exports/barrel policy: **Barrel exports used.** `packages/types/src/index.ts` exports all interfaces from a single file. No barrel exports found in web or api yet.

### 4) Error and Logging Conventions

- Error strategy by layer:
  - **API:** tRPC error formatting in `src/trpc.ts` (Zod-based). Sentry error handler at Express level. No structured error classes yet.
  - **Web:** No explicit error boundaries configured yet. Sentry client config exists.
  - **AI Service:** No error handling patterns established yet (all endpoints are stubs).
- Logging style and required context fields:
  - **API:** pino logger with pretty-print transport (dev). No structured logging configuration for production.
  - **Web:** No client-side logging library configured (Sentry for errors only).
  - **AI Service:** loguru is a dependency but no logging calls found in route files.
- Sensitive-data redaction rules: **Not established.** No evidence of PII/secret redaction in any layer.

### 5) Testing Conventions

- Test file naming/location rule: **Not established.** No test files exist anywhere in the monorepo.
- Mocking strategy norm: **Not established.**
- Coverage expectation: **Not configured.**

### 6) Evidence

- `eslint.base.json` — base lint rules
- `apps/web/.eslintrc.json` — web-specific lint rules
- `apps/web/tsconfig.json` — path alias configuration
- `packages/types/src/index.ts` — barrel export pattern
- `apps/api/src/trpc.ts` — tRPC error formatting
- `apps/api/src/index.ts` — pino logger setup
- `.env.example` — env var naming convention
- `CONTRIBUTING.md` — branch naming and commit conventions
