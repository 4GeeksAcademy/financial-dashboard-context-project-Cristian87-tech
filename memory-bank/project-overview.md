# Product overview and current state

**Status snapshot:** 2026-11-13
**Evidence standard:** Statements below describe checked-in code, project documentation, or commands run during this work. Scope claims to the cited evidence; absence of a feature in the cited files is not proof that it is absent from every deployment or external system.

## Product overview

This repository contains a financial metrics dashboard. The React interface requests financial movements from a FastAPI service, calculates summary KPIs and monthly aggregates in the frontend, and renders dashboard cards and charts.

The inspected `/api/metrics` implementation generates seeded mock movements in memory. The cited files do not establish whether other deployments or external systems provide persistent data. The product behavior described here is evidenced by:

- [`README.md`](../README.md): describes the financial metrics dashboard and the local service URLs.
- [`backend/app/routes.py`](../backend/app/routes.py): defines the movement and metrics models, mock-data generator, filters, and `/api/metrics` endpoints.
- [`frontend/src/App.tsx`](../frontend/src/App.tsx): fetches `/api/metrics`, coordinates loading/error state, and renders the dashboard.
- [`frontend/src/lib/financial-utils.ts`](../frontend/src/lib/financial-utils.ts): calculates KPI totals, profit percentage, and monthly chart data.
- [`frontend/src/components/dashboard/`](../frontend/src/components/dashboard/): contains the dashboard presentation components.

## Technology stack

| Area | Technologies and role | Repository evidence |
|---|---|---|
| Languages | Python 3.13; TypeScript/TSX; CSS | [`backend/Dockerfile`](../backend/Dockerfile), [`frontend/tsconfig.app.json`](../frontend/tsconfig.app.json), [`frontend/src/`](../frontend/src/) |
| Backend | FastAPI for HTTP routes and Pydantic response/data models; Uvicorn ASGI server | [`backend/requirements.txt`](../backend/requirements.txt), [`backend/app/main.py`](../backend/app/main.py), [`backend/app/routes.py`](../backend/app/routes.py) |
| Frontend | React 19 with Vite; TypeScript | [`frontend/package.json`](../frontend/package.json), [`frontend/vite.config.ts`](../frontend/vite.config.ts) |
| UI and visualization | Tailwind CSS, Recharts, Lucide React, `clsx`, `tailwind-merge`, and `class-variance-authority` | [`frontend/package.json`](../frontend/package.json), [`frontend/src/`](../frontend/src/) |
| Containers and local development | Dockerfiles for backend/frontend, Docker Compose; Vite development server and `/api` proxy | [`docker-compose.yml`](../docker-compose.yml), [`backend/Dockerfile`](../backend/Dockerfile), [`frontend/Dockerfile`](../frontend/Dockerfile), [`README.md`](../README.md) |
| Testing and quality tooling | pytest/pytest-cov/httpx; Vitest; ESLint; TypeScript compiler | [`backend/requirements.txt`](../backend/requirements.txt), [`frontend/package.json`](../frontend/package.json) |

Dependency versions and scripts should be read from the manifests themselves; this document intentionally avoids duplicating a version inventory that can become stale.

## Current status

### Working and verified

- The backend exposes `/health` and financial metrics endpoints including `/api/metrics`; its route tests cover filters and response behavior.
- CORS is configured through `CORS_ORIGINS`, defaulting to `http://localhost:5173` for local development. Compose forwards the setting to the backend.
- Focused backend test run: `cd backend && pytest tests/test_routes.py -q` — **22 passed** on 2026-11-13.
- Frontend KPI and monthly aggregation functions have Vitest coverage, including the empty-input case. The tests exist, but they were not successfully executed in this environment (see known gaps).

Evidence: [`backend/app/main.py`](../backend/app/main.py), [`backend/app/routes.py`](../backend/app/routes.py), [`backend/tests/test_routes.py`](../backend/tests/test_routes.py), [`docker-compose.yml`](../docker-compose.yml), [`frontend/src/lib/financial-utils.test.ts`](../frontend/src/lib/financial-utils.test.ts).

### Known gaps and unverified behavior

- **Debugger configuration:** `backend/Dockerfile` starts `debugpy` on `0.0.0.0:5678`, and `docker-compose.yml` publishes port `5678`. The repository does not document a separate production deployment configuration, so the deployment exposure cannot be determined from these files alone.
- **CORS configuration:** the checked-in default is `http://localhost:5173`, with `CORS_ORIGINS` configurable through Compose. No production origin value is specified in the checked-in configuration cited here.
- **Generated data:** the inspected `generate_mock_movements()` implementation creates seeded in-memory records. This documents that endpoint's data source; it does not establish the absence of other data systems outside the inspected code.
- **Frontend validation pending:** the focused Vitest command did not run because `vitest` was unavailable in the installed frontend dependencies. `npm ci` could not complete because of filesystem permissions. Frontend tests, lint, and build therefore remain unverified in this snapshot.
- **Integration-test status not established:** the backend route tests and frontend utility tests cited here are separate. No claim is made here about whether another integration test exists elsewhere in the repository.

Evidence: [`backend/Dockerfile`](../backend/Dockerfile), [`docker-compose.yml`](../docker-compose.yml), [`backend/app/routes.py`](../backend/app/routes.py), [`backend/requirements.txt`](../backend/requirements.txt), [`frontend/package.json`](../frontend/package.json), [`backend/tests/test_routes.py`](../backend/tests/test_routes.py), [`frontend/src/lib/financial-utils.test.ts`](../frontend/src/lib/financial-utils.test.ts).

### Pending checks (not a roadmap)

- The production values and network exposure for `CORS_ORIGINS` and `debugpy` are not established by the checked-in local configuration.
- The frontend test, lint, and build results are unknown because those commands were not successfully run in this environment.

These are evidence limits, not product commitments, approved priorities, or a proposed roadmap.
