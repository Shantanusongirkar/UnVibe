# Testing Patterns

## Core Sections (Required)

### 1) Test Stack and Commands

- Primary test framework: **None configured.** No test runner is declared in any `package.json` or in the AI service.
  - `turbo.json` defines a `test` pipeline task (depends on `^build`, cached) but no actual test script exists.
  - No test files exist anywhere in the monorepo.
- Assertion/mocking tools: **None selected.**
- Commands:
  ```bash
  pnpm test                 # Defined in turbo.json pipeline, but no test runners configured
  ```

**No test commands are functional.** The `test` script in `turbo.json` is a placeholder.

### 2) Test Layout

- Test file placement pattern: **Not established.** No test files exist.
- Naming convention: **Not established.**
- Setup files and where they run: **None.**

### 3) Test Scope Matrix

| Scope | Covered? | Typical target | Notes |
|-------|----------|----------------|-------|
| Unit | No | — | No test framework, no test files anywhere |
| Integration | No | — | No integration test setup |
| E2E | No | — | No Playwright, Cypress, or other E2E tooling |

**The entire project has zero tests.**

### 4) Mocking and Isolation Strategy

- Main mocking approach: **Not established.**
- Isolation guarantees: **Not established.**
- Common failure mode in tests: **N/A** — no tests exist.

### 5) Coverage and Quality Signals

- Coverage tool + threshold: **None configured.**
- Current reported coverage: **0%** — no tests exist.
- Known gaps/flaky areas: **All areas are gaps.** The entirety of the codebase is untested.

### 6) Evidence

- `turbo.json` — `test` pipeline task defined but no underlying script
- `apps/web/package.json` — no `test` script
- `apps/api/package.json` — no `test` script
- `apps/ai-service/requirements.txt` — no test dependencies
- Glob search for `*.test.*`, `*.spec.*`, `__tests__/`, `tests/` — all returned empty
