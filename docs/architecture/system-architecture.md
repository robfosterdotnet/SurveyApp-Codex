# System Architecture Draft

## High-Level Layers
1. **Client (React + shadcn/ui)** – delivers survey builder, respondent UI, and admin dashboard. Communicates via REST + WebSocket to FastAPI and handles optimistic updates for common interactions.
2. **API (FastAPI)** – orchestrates survey lifecycle, authentication, analytics aggregation, and background processing through Celery/worker placeholder. Exposes modular routers secured by JWT + RBAC middleware.
3. **Data (Postgres + Redis)** – Postgres stores surveys, questions, responses, recipients, analytics snapshots; Redis supports caching, WebSocket presence, and Celery queues. Future OLAP store can be added for heavy reporting.

## Module Breakdown
- `frontend/src/features` – survey builder, distribution console, dashboard widgets, respondent shell.
- `frontend/src/services` – REST/WS clients, auth, feature flags, translation loader.
- `frontend/src/shared` – UI primitives, hooks (e.g., `useAutosave`), validation schemas.
- `backend/app/api` – routers grouped by domain (surveys, responses, recipients, analytics, admin, auth).
- `backend/app/core` – settings, security, dependency injection, email adapters.
- `backend/app/models` – SQLAlchemy models mapping to Postgres plus Pydantic schemas for IO.
- `backend/app/workers` – Celery tasks for distribution, CSV exports, anomaly detection, warehouse sync.

## Data Flow
1. **Survey authoring**
   - Designer defines survey in frontend → `POST /surveys` persists config → Postgres tables: `surveys`, `sections`, `questions`, `branch_rules`.
   - Autosave uses `PATCH /surveys/:id` debounced updates; optimistic UI caches in IndexedDB for offline protection.
2. **Distribution**
   - Admin uploads recipients → `POST /surveys/:id/distributions` stores metadata + generated token hashed with HMAC secret.
   - Celery worker dequeues batch → calls email provider API with template, custom fields, and per-link expiry timestamp.
3. **Response capture**
   - Respondent opens `/respond/:token` → backend validates token, upserts record in `responses` + `response_answers`.
   - Autosave via `PATCH /responses/:id` includes heartbeat timestamp to enforce session timeout; file uploads stored in object storage bucket with signed URLs.
   - Completion triggers `POST /responses/:id/complete`, locking record and notifying anomaly detection worker.
4. **Analytics + monitoring**
   - Admin dashboard hits `/surveys/:id/metrics` (REST) for baseline numbers and subscribes to `/ws/surveys/:id/metrics` channel for incremental updates published by worker after each response ingestion.
   - Scheduled snapshot job aggregates into `analytics_snapshots` table and replicates to warehouse schema nightly.

## Security & Compliance
- JWT-issued access tokens for designers/admins; short-lived (15 min) with refresh flow. Respondent tokens remain opaque HMAC strings and only support survey access.
- RBAC enforced via middleware referencing `workspace_roles` table; audit log table captures actor, action, entity, payload hash.
- Encryption at rest via managed Postgres/Redis plus encrypted object storage; sensitive fields (PII) optionally column-level encrypted using AES (libsodium wrapper).
- Secrets sourced from `.env` in dev, centralized secrets manager in prod; rotation reminders tracked in `docs/security.md`.

## Deployment Notes
- Containerize frontend and backend separately; share `.env` contract stored in `deploy/` (to be created later) and include migrations image.
- Use reverse proxy (e.g., Traefik or Nginx) for TLS offload and route `/api` + `/ws` to FastAPI service; static assets served via CDN.
- Background workers consume from Redis queue for bulk email + analytics rollups; Redis cluster also used for WebSocket pub/sub (via FastAPI + Uvicorn with `broadcast`).
- CI/CD: GitHub Actions runs lint/test/build, builds containers, pushes to registry, and triggers deploy via IaC (Terraform module placeholder).

## Observability
- Structured logging (OpenTelemetry) with trace IDs passed from frontend using `x-request-id`.
- Metrics: response count, completion % per survey, API latency, worker queue depth, email delivery success, snapshot job duration.
- Distributed tracing spans across client, API, worker jobs; sample 20% of respondent sessions.
- Alerts for surveys nearing closing date with <70% completion, worker queue backlog > 500 jobs, error rate > 2% over 5 minutes.

## Scalability & Resilience
- FastAPI pods scale horizontally based on CPU and WebSocket connections; sticky sessions avoided by storing session state in Redis.
- Postgres uses read replica for analytics queries; heavy exports executed against replica to avoid impacting OLTP workload.
- Feature flags (LaunchDarkly placeholder) guard experimental builder components and new question types.
- Rate limiting middleware ensures per-IP/per-token thresholds to mitigate abuse; WAF handles bot traffic on respondent endpoints.

## Outstanding Decisions
- Pick between SQLModel vs. SQLAlchemy ORM for FastAPI models (leaning SQLAlchemy for advanced features).
- Choose state management (Redux Toolkit vs. Zustand) for complex survey builder interactions and offline caching.
- Determine hosting (managed container service vs. Kubernetes) and secrets manager.
- Select email provider (Resend, Postmark, SES) and virus scanning approach for file uploads.
