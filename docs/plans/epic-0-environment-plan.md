# Plan – Epic 0: Environment & Platform Foundations

Linked epic: `docs/requirements/implementation-epics.md` (Epic 0).

## Objectives
1. Provide a reproducible Docker/Compose environment (frontend, backend, workers, Postgres, Redis) for local and Raspberry Pi 5 deployments.
2. Establish environment/secrets management with `.env.local` and vault-ready patterns.
3. Instrument telemetry (OpenTelemetry, structured logs, metrics/alerts) usable by downstream epics.
4. Configure CI pipelines enforcing lint/test/coverage baselines (ESLint+Prettier, Vitest, Pytest, ≥80% lines).

## Workstreams & Tasks

### WS1 – Docker & Compose Scaffolding
| Task | Description | Owner | Dependencies | Acceptance |
| --- | --- | --- | --- | --- |
| WS1.1 | Define base Dockerfiles for frontend (Next.js) and backend (FastAPI + SQLAlchemy) | Eng | None | `docker build` succeeds for both services |
| WS1.2 | Create `docker-compose.yml` running frontend, backend, Postgres, Redis, Celery worker | Eng | WS1.1 | `docker compose up` serves app on port 3000/8000 |
| WS1.3 | Document Raspberry Pi 5 setup (Docker CE arm64 + Compose plugin) | DevOps | WS1.2 | Guide in `docs/architecture/system-architecture.md` + `README.md` |
| WS1.4 | Add seed scripts for initializing Postgres schemas/migrations | Backend | WS1.2 | `docker compose run backend alembic upgrade head` works |

### WS2 – Environment & Secrets Management
| Task | Description | Owner | Dependencies | Acceptance |
| --- | --- | --- | --- | --- |
| WS2.1 | Define `.env.example` capturing required vars for frontend/backend/workers | Eng | WS1 | Devs can copy to `.env.local` and run services |
| WS2.2 | Integrate dotenv loading in Next.js + FastAPI configs; document precedence | Eng | WS2.1 | Services read from `.env.local` without runtime warnings |
| WS2.3 | Outline vault integration (prod secrets) and local overrides | DevOps | WS2.1 | Architecture doc updated with secret flow diagrams |
| WS2.4 | Harden secrets in Compose (`.env` references, no secrets committed) | Eng | WS1 | `git status` never shows secrets; Compose env vars load |

### WS3 – Telemetry & Observability
| Task | Description | Owner | Dependencies | Acceptance |
| --- | --- | --- | --- | --- |
| WS3.1 | Install OpenTelemetry SDKs (frontend, backend) with basic spans | Eng | WS1 | Sample spans visible in console/logs |
| WS3.2 | Configure structured logging with correlation IDs | Backend | WS3.1 | Logs include trace/span IDs and user/request context |
| WS3.3 | Expose metrics endpoint (FastAPI) tracking autosave, responses, anomalies | Backend | WS1 | `/metrics` scrape-ready; sample data displayed |
| WS3.4 | Define alert thresholds (completion <60%, autosave errors spike) in docs/observability.md | DevOps | WS3.3 | Alert runbooks documented |

### WS4 – CI/CD & Testing Guardrails
| Task | Description | Owner | Dependencies | Acceptance |
| --- | --- | --- | --- | --- |
| WS4.1 | Configure GitHub Actions (or chosen CI) to run lint/test on PRs | Eng | Repo access | CI job green for sample PR |
| WS4.2 | Add coverage reporting (Vitest + Pytest) with ≥80% gate | Eng | WS4.1 | CI fails when coverage <80% |
| WS4.3 | Integrate MSW for frontend tests and update test scaffolding | Frontend | WS1 | Sample test hitting mocked API passes |
| WS4.4 | Document “Getting Started” steps in `README.md` including CI expectations | Eng | WS4.1 | New devs can follow steps to contribute |

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
