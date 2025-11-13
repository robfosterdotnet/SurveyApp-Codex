# System Architecture Draft

## High-Level Layers
1. **Client (React + shadcn/ui)** – delivers survey builder, respondent UI, and admin dashboard. Communicates via REST + WebSocket to FastAPI.
2. **API (FastAPI)** – orchestrates survey lifecycle, authentication, analytics aggregation, and background processing through Celery/worker placeholder.
3. **Data (Postgres)** – stores surveys, questions, responses, recipients, analytics snapshots. Future OLAP store can be added for heavy reporting.

## Module Breakdown
- `frontend/src/features` – survey builder, distribution console, dashboard widgets.
- `frontend/src/services` – REST/WS clients, auth, feature flags.
- `backend/app/api` – routers grouped by domain (surveys, responses, admin, auth).
- `backend/app/core` – settings, security, dependency injection.
- `backend/app/models` – SQLModel or Pydantic models mapping to Postgres.

## Data Flow
1. Designer defines survey in frontend → FastAPI `POST /surveys` persists config → Postgres.
2. Admin triggers distribution → backend generates recipient rows + tokens → email provider sends unique URLs `/respond/:token`.
3. Respondent answers → UI autosaves via `PATCH /responses/:id` until complete → closing date enforced.
4. Admin dashboard subscribes to WebSocket `/ws/surveys/:id/metrics` for live aggregates.

## Deployment Notes
- Containerize frontend and backend separately; share `.env` contract stored in `deploy/` (to be created later).
- Use reverse proxy (e.g., Traefik or Nginx) for TLS offload and route `/api` + `/ws` to FastAPI service.
- Background workers consume from Redis queue for bulk email + analytics rollups.

## Observability
- Structured logging (OpenTelemetry) with trace IDs passed from frontend.
- Metrics: response count, completion % per survey, API latency, worker queue depth.
- Alerts for surveys nearing closing date with <70% completion.

## Outstanding Decisions
- Pick between SQLModel vs. SQLAlchemy ORM for FastAPI models.
- Choose state management (Redux Toolkit, Zustand, Recoil) for complex survey builder interactions.
- Determine hosting (managed container service vs. Kubernetes) and secrets manager.
