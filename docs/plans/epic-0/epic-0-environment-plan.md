# Plan – Epic 0: Environment & Platform Foundations

Linked epic: `docs/requirements/implementation-epics.md` (Epic 0).

## Objectives
1. Provide a reproducible Docker/Compose environment (frontend, backend, workers, Postgres, Redis) for local and Raspberry Pi 5 deployments.
2. Establish environment/secrets management with `.env.local` and vault-ready patterns.
3. Instrument telemetry (OpenTelemetry, structured logs, metrics/alerts) usable by downstream epics.
4. Configure CI pipelines enforcing lint/test/coverage baselines (ESLint+Prettier, Vitest, Pytest, ≥80% lines).

## Prerequisites (Raspberry Pi 5)
- Raspberry Pi OS 64-bit updated via `sudo apt update && sudo apt upgrade`.
- Build tools: `build-essential`, `python3-dev`, `pkg-config`, `libpq-dev`.
- Docker CE (arm64) + Docker Compose plugin installed, service enabled, and user added to `docker` group.
- Git, Node 20+, npm, and Python 3.11+ available for local tooling tasks.
- Vault/secret manager CLI if validating WS2 vault flows on-device.
- Connectivity to telemetry collectors (OpenTelemetry/Prometheus endpoints) if running locally.

## Workstreams & Tasks

### WS1 – Docker & Compose Scaffolding
| Task | Description | Owner | Dependencies | Acceptance |
| --- | --- | --- | --- | --- |
| WS1.1 | Define base Dockerfiles for frontend (Next.js) and backend (FastAPI + SQLAlchemy) | Eng | None | `docker build` succeeds for both services |
| WS1.2 | Create `docker-compose.yml` running frontend, backend, Postgres, Redis, Celery worker | Eng | WS1.1 | `docker compose up` serves app on port 3000/8000 |
| WS1.3 | Document Raspberry Pi 5 setup (Docker CE arm64 + Compose plugin) | DevOps | WS1.2 | Guide in `docs/architecture/system-architecture.md` + `README.md` |
| WS1.4 | Add seed scripts for initializing Postgres schemas/migrations | Backend | WS1.2 | `docker compose run backend alembic upgrade head` works |

#### WS1 Task Details
- **WS1.1 – Base Dockerfiles**
  1. Create `src/frontend/Dockerfile` with a multi-stage build optimized for Vite/Next.js:
     ```Dockerfile
     # src/frontend/Dockerfile
     FROM node:20-alpine AS deps
     WORKDIR /app
     COPY package.json package-lock.json ./
     RUN npm ci

     FROM node:20-alpine AS builder
     WORKDIR /app
     COPY --from=deps /app/node_modules ./node_modules
     COPY . .
     RUN npm run build

     FROM node:20-alpine AS runner
     WORKDIR /app
     ENV NODE_ENV=production
     COPY --from=builder /app/.next ./.next
     COPY --from=builder /app/public ./public
     COPY package.json package-lock.json ./
     RUN npm ci --omit=dev
     EXPOSE 3000
     CMD ["npm","run","start"]
     ```
  2. Create `src/backend/Dockerfile` for FastAPI + Uvicorn/Gunicorn:
     ```Dockerfile
     # src/backend/Dockerfile
     FROM python:3.11-slim
     ENV PYTHONDONTWRITEBYTECODE=1
     ENV PYTHONUNBUFFERED=1
     WORKDIR /app
     RUN apt-get update && apt-get install -y build-essential libpq-dev && rm -rf /var/lib/apt/lists/*
     COPY requirements.txt .
     RUN pip install --no-cache-dir -r requirements.txt
     COPY . .
     EXPOSE 8000
     CMD ["uvicorn","app.main:app","--host","0.0.0.0","--port","8000"]
     ```
  3. Run `docker build -t surveyapp-frontend -f src/frontend/Dockerfile src/frontend` and `docker build -t surveyapp-backend -f src/backend/Dockerfile src/backend` on amd64 + Raspberry Pi 5 (arm64) to verify cross-architecture success.

- **WS1.2 – Compose Stack**
  1. Create `docker-compose.yml` at repo root:
     ```yaml
     version: "3.9"
     services:
       postgres:
         image: postgres:15
         environment:
           POSTGRES_DB: surveydb
           POSTGRES_USER: survey
           POSTGRES_PASSWORD: changeme
         volumes:
           - postgres-data:/var/lib/postgresql/data
       redis:
         image: redis:7
       backend:
         build: ./src/backend
         env_file: .env.local
         command: uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
         volumes:
           - ./src/backend:/app
         depends_on:
           - postgres
           - redis
       worker:
         build: ./src/backend
         command: celery -A app.worker worker -l info
         env_file: .env.local
         depends_on:
           - backend
           - redis
       frontend:
         build: ./src/frontend
         env_file: .env.local
         volumes:
           - ./src/frontend:/app
         ports:
           - "3000:3000"
         depends_on:
           - backend
     volumes:
       postgres-data:
     ```
  2. Add `.dockerignore` files in frontend/backend to reduce context (node_modules, build, .venv).
  3. Validate with `docker compose up --build` and confirm http://localhost:3000 (frontend) and http://localhost:8000/docs (backend) respond.
  4. Document service port mapping and health-check commands inside `docs/architecture/system-architecture.md`.

- **WS1.3 – Raspberry Pi 5 Setup**
  1. Capture the installation flow in `docs/architecture/system-architecture.md`:
     - Update OS (`sudo apt update && sudo apt upgrade`).
     - Install dependencies (`sudo apt install -y build-essential python3-dev pkg-config libpq-dev`).
     - Install Docker CE arm64 and Compose plugin per https://docs.docker.com/engine/install/debian/ .
     - Add the developer to `docker` group (`sudo usermod -aG docker $USER`).
     - Validate `docker run hello-world`.
  2. In `README.md`, add a “Raspberry Pi 5 Notes” section summarizing the above plus Pi-specific flags (e.g., `DOCKER_DEFAULT_PLATFORM=linux/arm64` when building from amd64 host).

- **WS1.4 – Database Seed Scripts**
  1. Create `src/backend/migrations/env.py` + Alembic `versions/` folder with base migration (surveys, sections, questions tables).
  2. Create `scripts/bootstrap-db.sh`:
     ```bash
     #!/usr/bin/env bash
     set -euo pipefail
     docker compose run --rm backend alembic upgrade head
     docker compose run --rm backend python seed/sample_data.py
     ```
  3. Include `seed/sample_data.py` to insert default designer account + sample survey metadata.
  4. Document execution in `docs/architecture/system-architecture.md` and ensure `chmod +x scripts/bootstrap-db.sh`.

### WS2 – Environment & Secrets Management
| Task | Description | Owner | Dependencies | Acceptance |
| --- | --- | --- | --- | --- |
| WS2.1 | Define `.env.example` capturing required vars for frontend/backend/workers | Eng | WS1 | Devs can copy to `.env.local` and run services |
| WS2.2 | Integrate dotenv loading in Next.js + FastAPI configs; document precedence | Eng | WS2.1 | Services read from `.env.local` without runtime warnings |
| WS2.3 | Outline vault integration (prod secrets) and local overrides | DevOps | WS2.1 | Architecture doc updated with secret flow diagrams |
| WS2.4 | Harden secrets in Compose (`.env` references, no secrets committed) | Eng | WS1 | `git status` never shows secrets; Compose env vars load |

#### WS2 Task Details
- **WS2.1 – `.env.example`**
  1. Create `.env.example` at the repo root with comments:
     ```
     # Frontend
     NEXT_PUBLIC_API_BASE_URL=http://localhost:8000
     NEXT_PUBLIC_OTEL_EXPORTER_URL=http://localhost:4318

     # Backend
     DATABASE_URL=postgresql+psycopg://survey:changeme@postgres:5432/surveydb
     REDIS_URL=redis://redis:6379/0
     SECRET_KEY=replace-me
     OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector:4317

     # Workers
     CELERY_BROKER_URL=redis://redis:6379/1
     CELERY_RESULT_BACKEND=redis://redis:6379/2
     ```
  2. Add instructions in the file header to copy into `.env.local`: `cp .env.example .env.local`.

- **WS2.2 – Dotenv Integration**
  1. Frontend: install `npm install dotenv --save-dev` and create `src/frontend/next.config.js`:
     ```js
     const dotenv = require('dotenv');
     dotenv.config({ path: '.env.local' });

     const nextConfig = {
       reactStrictMode: true,
       env: {
         NEXT_PUBLIC_API_BASE_URL: process.env.NEXT_PUBLIC_API_BASE_URL,
       },
     };
     module.exports = nextConfig;
     ```
  2. Backend: in `src/backend/app/core/settings.py`, use `pydantic-settings`:
     ```python
     from pydantic_settings import BaseSettings

     class Settings(BaseSettings):
         database_url: str
         redis_url: str
         secret_key: str

         class Config:
             env_file = ".env.local"

     settings = Settings()
     ```
  3. Document env precedence in `README.md` (“`.env.local` overrides `.env` and Docker secrets”).

- **WS2.3 – Vault Integration Outline**
  1. Create `docs/architecture/secrets.md` describing environments:
     - Local uses `.env.local`.
     - Staging/Prod pull from Vault path `kv/surveyapp/<env>` with CLI snippet:
       ```bash
       vault kv get -format=json kv/surveyapp/prod | jq -r '.data.data | to_entries[] | "\(.key)=\(.value)"' > .env.runtime
       ```
  2. Include diagram (Mermaid) showing flow from vault to Docker secrets.
  3. Note rotation policy and fallback instructions for offline dev.

- **WS2.4 – Secret Hardening**
  1. Update `docker-compose.yml` services to reference env files:
     ```yaml
     env_file:
       - .env.local
     ```
  2. Add `.env.local`, `.env.runtime`, and `*.secret` patterns to `.gitignore`.
  3. Configure a pre-commit hook (`.githooks/pre-commit`) running `git secrets --scan` and update `README.md` with install instructions.

### WS3 – Telemetry & Observability
| Task | Description | Owner | Dependencies | Acceptance |
| --- | --- | --- | --- | --- |
| WS3.1 | Install OpenTelemetry SDKs (frontend, backend) with basic spans | Eng | WS1 | Sample spans visible in console/logs |
| WS3.2 | Configure structured logging with correlation IDs | Backend | WS3.1 | Logs include trace/span IDs and user/request context |
| WS3.3 | Expose metrics endpoint (FastAPI) tracking autosave, responses, anomalies | Backend | WS1 | `/metrics` scrape-ready; sample data displayed |
| WS3.4 | Define alert thresholds (completion <60%, autosave errors spike) in docs/observability.md | DevOps | WS3.3 | Alert runbooks documented |

#### WS3 Task Details
- **WS3.1 – OpenTelemetry Instrumentation**
  1. Frontend: `npm install @opentelemetry/api @opentelemetry/sdk-trace-web @opentelemetry/exporter-trace-otlp-http`.
  2. Initialize in `src/frontend/src/lib/telemetry.ts`:
     ```ts
     import { WebTracerProvider } from '@opentelemetry/sdk-trace-web';
     import { BatchSpanProcessor } from '@opentelemetry/sdk-trace-base';
     import { OTLPTraceExporter } from '@opentelemetry/exporter-trace-otlp-http';

     const provider = new WebTracerProvider();
     provider.addSpanProcessor(
       new BatchSpanProcessor(
         new OTLPTraceExporter({ url: process.env.NEXT_PUBLIC_OTEL_EXPORTER_URL })
       )
     );
     provider.register();
     ```
  3. Backend: `pip install opentelemetry-sdk opentelemetry-exporter-otlp opentelemetry-instrumentation-fastapi`.
     ```python
     from opentelemetry.instrumentation.fastapi import FastAPIInstrumentor
     from opentelemetry.sdk.trace import TracerProvider
     from opentelemetry.sdk.trace.export import BatchSpanProcessor
     from opentelemetry.exporter.otlp.proto.http.trace_exporter import OTLPSpanExporter

     provider = TracerProvider()
     provider.add_span_processor(BatchSpanProcessor(OTLPSpanExporter(endpoint=settings.otel_endpoint)))
     trace.set_tracer_provider(provider)
     FastAPIInstrumentor.instrument_app(app, tracer_provider=provider)
     ```
  4. Verify spans using `otel-cli status` or by hitting `/health` endpoint and inspecting console output.

- **WS3.2 – Structured Logging**
  1. Install `pip install structlog`.
  2. Configure logger in `app/core/logging.py`:
     ```python
     import structlog
     import uuid

     structlog.configure(processors=[structlog.processors.TimeStamper(), structlog.processors.JSONRenderer()])
     logger = structlog.get_logger()

     async def log_request(request, call_next):
         request_id = request.headers.get("x-request-id", str(uuid.uuid4()))
         response = await call_next(request)
         logger.info("request.complete", request_id=request_id, path=request.url.path, status=response.status_code)
         return response
     ```
  3. Add middleware to FastAPI app and include `trace.get_current_span().get_span_context().trace_id` to logs for correlation.

- **WS3.3 – Metrics Endpoint**
  1. Install `pip install prometheus-fastapi-instrumentator`.
  2. In `app/main.py`:
     ```python
     from prometheus_fastapi_instrumentator import Instrumentator

     instrumentator = Instrumentator().add(
         metrics.autosave_failures(), metrics.response_latency(), metrics.branching_anomalies()
     )
     instrumentator.instrument(app).expose(app, include_in_schema=False)
     ```
  3. Define custom metrics in `app/telemetry/metrics.py` using `prometheus_client.Counter/Histogram`.
  4. Run `curl http://localhost:8000/metrics` to confirm autosave/response/anomaly metrics appear.

- **WS3.4 – Alert Thresholds**
  1. Create `docs/observability.md` with sections:
     - Completion rate <60% for 10 minutes → Slack alert.
     - Autosave errors >5/min → Pager rotation.
     - Branch anomaly counts >3σ baseline.
  2. Provide sample PromQL queries:
     ```text
     (sum(rate(survey_autosave_error_total[5m])) by (survey_id)) > 5
     ```
  3. Include runbook steps (how to check logs, restart worker, notify designer).

### WS4 – CI/CD & Testing Guardrails
| Task | Description | Owner | Dependencies | Acceptance |
| --- | --- | --- | --- | --- |
| WS4.1 | Configure GitHub Actions (or chosen CI) to run lint/test on PRs | Eng | Repo access | CI job green for sample PR |
| WS4.2 | Add coverage reporting (Vitest + Pytest) with ≥80% gate | Eng | WS4.1 | CI fails when coverage <80% |
| WS4.3 | Integrate MSW for frontend tests and update test scaffolding | Frontend | WS1 | Sample test hitting mocked API passes |
| WS4.4 | Document “Getting Started” steps in `README.md` including CI expectations | Eng | WS4.1 | New devs can follow steps to contribute |

#### WS4 Task Details
- **WS4.1 – GitHub Actions CI**
  1. Create `.github/workflows/ci.yml`:
     ```yaml
     name: CI
     on:
       pull_request:
         branches: [main]
     jobs:
       build:
         runs-on: ubuntu-latest
         services:
           postgres:
             image: postgres:15
             env:
               POSTGRES_DB: surveydb
               POSTGRES_USER: survey
               POSTGRES_PASSWORD: changeme
             ports: ["5432:5432"]
         steps:
           - uses: actions/checkout@v4
           - uses: actions/setup-node@v4
             with:
               node-version: 20
           - uses: actions/setup-python@v5
             with:
               python-version: '3.11'
           - name: Install frontend deps
             working-directory: src/frontend
             run: npm ci
           - name: Install backend deps
             working-directory: src/backend
             run: pip install -r requirements.txt
           - name: Run lint
             working-directory: src/frontend
             run: npm run lint
           - name: Run Pytest
             working-directory: src/backend
             run: pytest
     ```
  2. Add status badge to `README.md`.

- **WS4.2 – Coverage Gates**
  1. Frontend: configure Vitest coverage in `vitest.config.ts`:
     ```ts
     export default defineConfig({
       test: {
         coverage: {
           provider: 'v8',
           lines: 0.8,
         },
       },
     });
     ```
  2. Backend: add to `pyproject.toml`:
     ```toml
     [tool.pytest.ini_options]
     addopts = "--cov=app --cov-report=xml --cov-fail-under=80"
     ```
  3. Update CI workflow to run `npm run test -- --coverage` and `pytest` with coverage gate.
  4. Upload coverage artifacts using `actions/upload-artifact`.

- **WS4.3 – MSW Integration**
  1. Install MSW: `npm install msw @testing-library/react @testing-library/jest-dom -D`.
  2. Create `src/frontend/src/mocks/handlers.ts` and `src/frontend/src/mocks/server.ts`.
  3. Configure Vitest setup file `src/frontend/src/test/setup.ts`:
     ```ts
     import { beforeAll, afterAll, afterEach } from 'vitest';
     import { server } from '../mocks/server';

     beforeAll(() => server.listen());
     afterEach(() => server.resetHandlers());
     afterAll(() => server.close());
     ```
  4. Write a sample test `tests/features/surveys/builder.spec.ts` that mocks `/api/surveys` and asserts autosave indicator renders. Ensure it runs via `npm run test`.

- **WS4.4 – README Getting Started**
  1. Add sections:
     - “Quick Start” (clone repo, `cp .env.example .env.local`, `docker compose up --build`).
     - “Local Commands” (`npm run dev`, `npm run lint`, `pytest`, `scripts/bootstrap-db.sh`).
     - “CI Expectations” describing branch naming, lint/test/coverage gates, required status checks.
  2. Include troubleshooting tips (Docker memory, port conflicts) and link to Raspberry Pi appendix.

## Sequencing & Milestones
1. **Milestone A – Compose Up:** Complete WS1.1–WS1.3 so engineers can run full stack locally (target Week 1).
2. **Milestone B – Secrets Ready:** Deliver WS2 tasks enabling standardized env loading (target Week 1.5).
3. **Milestone C – Telemetry Baseline:** Finish WS3 tasks so downstream epics inherit tracing/logging (target Week 2).
4. **Milestone D – CI Guardrails:** Complete WS4 tasks, enforcing lint/test/coverage on PRs (target Week 2.5).

## Risks & Mitigations
- **Pi-specific Docker quirks:** run validation on actual Raspberry Pi 5 early; document architecture-specific flags.
- **Telemetry overhead/perf:** start with sampling to avoid dev slowdown; fine-tune later.
- **CI flakiness:** use MSW + deterministic fixtures; add retries for integration steps.

## Exit Criteria
- `docker compose up` spins up full stack with sample data in <5 minutes.
- `.env.example` and docs enable any new contributor to bootstrap within 15 minutes.
- Traces/logs/metrics available for use in subsequent epics; alert thresholds noted.
- CI pipeline running automatically with lint/test/coverage gates on every PR.
