# System Architecture Draft

## High-Level Layers
1. **Client (React + shadcn/ui)** – Next.js/Vite-driven frontend that delivers survey builder, respondent UI, and admin dashboard. Communicates via REST + WebSocket to FastAPI, emits OpenTelemetry spans, and uses IndexedDB-backed autosave caches for offline safety.
2. **API (FastAPI)** – orchestrates survey lifecycle, authentication, analytics aggregation, and background processing through Celery/worker container. Exposes modular routers secured by JWT + RBAC middleware, structured logging, and Prometheus-friendly metrics.
3. **Data (Postgres + Redis)** – Postgres stores surveys, questions, responses, recipients, analytics snapshots; Redis supports caching, WebSocket presence, Celery queues, and rate limiting. Future OLAP store can be added for heavy reporting or anomaly detection.

## Module Breakdown
- `frontend/src/features` – survey builder, distribution console, dashboard widgets, respondent shell.
- `frontend/src/services` – REST/WS clients, auth, feature flags, translation loader, OpenTelemetry initializers.
- `frontend/src/shared` – UI primitives, hooks (e.g., `useAutosave`), validation schemas, MSW test handlers.
- `backend/app/api` – routers grouped by domain (surveys, responses, recipients, analytics, admin, auth).
- `backend/app/core` – settings, security, dependency injection, email adapters, structlog/OpenTelemetry setup, Prometheus instrumentation.
- `backend/app/models` – SQLAlchemy models mapping to Postgres plus Pydantic schemas for IO.
- `backend/app/workers` – Celery tasks for distribution, CSV exports, anomaly detection, warehouse sync.
- `scripts/` – automation helpers (`bootstrap-db.sh`, Alembic commands, telemetry diagnostics).

## Local Development Topology

Epic 0 provisions a reproducible Docker/Compose environment targeting both amd64 laptops and Raspberry Pi 5:

```
docker-compose.yml
├── postgres (port 5432) – persisted to volume `postgres-data`
├── redis (port 6379) – cache, pub/sub, Celery broker
├── backend (FastAPI on 8000) – mounts `./src/backend`, reload enabled for dev
├── worker (Celery) – reuses backend image, runs `celery -A app.worker worker -l info`
└── frontend (Next.js on 3000) – mounts `./src/frontend`, runs `npm run dev`
```

- Dockerfiles live in `src/frontend/Dockerfile` and `src/backend/Dockerfile` using multi-stage Node 20 Alpine and Python 3.11 slim images respectively. Both support `docker build` on amd64 + arm64 (Pi).
- `.env.example` defines shared variables (API base URL, Postgres DSN, Redis URL, OTLP endpoints). Developers copy to `.env.local`; Compose references the file through `env_file`.
- Commands:
  - `cp .env.example .env.local`
  - `docker compose up --build`
  - `scripts/bootstrap-db.sh` (runs Alembic migrations + `seed/sample_data.py`)
- Raspberry Pi appendix (same file) details OS prep (`sudo apt update && sudo apt upgrade`), package installs (`build-essential python3-dev pkg-config libpq-dev`), Docker CE arm64 installation, and validation steps (`docker run hello-world`, `DOCKER_DEFAULT_PLATFORM=linux/arm64` when cross-building).

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

## Configuration & Secrets
- Dev-config lives in `.env.example` and copies into `.env.local`. It includes:
  ```
  NEXT_PUBLIC_API_BASE_URL=http://localhost:8000
  DATABASE_URL=postgresql+psycopg://survey:changeme@postgres:5432/surveydb
  REDIS_URL=redis://redis:6379/0
  SECRET_KEY=replace-me
  CELERY_BROKER_URL=redis://redis:6379/1
  ```
- Frontend loads env vars through `next.config.js` (`dotenv.config({ path: '.env.local' })`). Backend uses `pydantic-settings` with `env_file = ".env.local"` and exposes settings via dependency injection.
- Production secrets flow from Vault: `vault kv get -format=json kv/surveyapp/<env> | jq -r '.data.data | to_entries[] | "\(.key)=\(.value)"' > .env.runtime`. Deployment pipelines mount `.env.runtime` or Docker secrets into containers.
- `.gitignore` covers `.env.local`, `.env.runtime`, and `*.secret`. Git hooks enforce `git secrets --scan`. Compose never hardcodes credentials—only references env keys defined above.

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

### CI/CD Implementation
- `.github/workflows/ci.yml` runs on every PR: checkout, setup Node 20 and Python 3.11, install deps in `src/frontend` (npm ci) + `src/backend` (pip install), run `npm run lint`, `npm run test -- --coverage`, and `pytest --cov=app --cov-report=xml --cov-fail-under=80`.
- Workflow starts Postgres service via `services.postgres` and wires `DATABASE_URL` secrets. Coverage artifacts upload for inspection; badge referenced in `README.md`.
- Subsequent iterations add container build/push jobs once runtime code exists.

## Observability
- **Tracing:** Frontend installs `@opentelemetry/sdk-trace-web` + OTLP HTTP exporter pointed at `NEXT_PUBLIC_OTEL_EXPORTER_URL`. Backend uses `opentelemetry-instrumentation-fastapi` with `BatchSpanProcessor` + OTLP exporter; spans propagate via `traceparent` header.
- **Logging:** Structlog middleware injects `x-request-id`, `trace_id`, and user context into JSON logs; Celery workers emit correlation IDs so traces/logs align.
- **Metrics:** `/metrics` endpoint exposed via `prometheus-fastapi-instrumentator` plus custom counters/histograms (autosave errors, response latency, branching anomalies, worker queue depth). PromQL queries documented in `docs/observability.md`.
- **Alerts:** Completion rate <60% for 10 minutes, autosave errors >5/min, anomaly counts >3σ baseline, worker backlog >500, error rate >2%/5min. Runbooks describe verification steps (curl endpoints, check `kubectl logs`, notify design lead).
- **Client instrumentation:** Autosave hook emits telemetry events for success/failure; preview builder records branching validation results to correlate with backend validations.

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
