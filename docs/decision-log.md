# Decision Log

Tracking major decisions and unresolved questions for SurveyApp-Codex. Update entries as plans evolve and link back to the source docs or tickets.

## Confirmed Decisions

| Date | Area | Decision | Rationale | Status | References |
| --- | --- | --- | --- | --- | --- |
| 2025-11-13 | Architecture | React 18 + shadcn/ui frontend, FastAPI backend, Postgres datastore, background workers via Celery/Redis | Aligns with requirement for rich survey builder UI, Python expertise for API, and relational model for survey data | Confirmed | README.md; docs/architecture/system-architecture.md |
| 2025-11-13 | Repo Structure | Maintain parallel `docs/`, `src/frontend`, `src/backend`, `tests/` with docs housing requirements + user stories | Keeps planning assets centralized and mirrors runtime layout for predictable navigation | Confirmed | README.md; docs/README.md |
| 2025-11-13 | Functional Scope | Support branching surveys, multiple question types (choice, Likert, ranking, numeric, text, date, matrix, file upload), cloning, autosave, CSV exports | Captures designer/admin needs outlined in product requirements and user stories | Confirmed | docs/requirements/product-requirements.md; docs/user-stories.md |
| 2025-11-13 | Distribution Model | Generate unique expiring links per recipient with optional auth (email token/SSO) and deliver via external email provider | Enables respondent tracking while supporting low-friction login | Confirmed | docs/requirements/product-requirements.md |
| 2025-11-13 | Observability | Adopt OpenTelemetry tracing, structured logs, metrics on response volume/completion, alerts for low completion before closing date | Provides visibility for admins/operators and supports SLA/availability targets | Confirmed | docs/architecture/system-architecture.md |
| 2025-11-13 | Testing Practices | Use Vitest + React Testing Library for frontend, Pytest for backend (mirroring `tests/` tree) with ≥80% coverage, MSW for mocks, log flaky tests in `docs/test-notes.md` | Aligns with AGENTS guidelines and keeps deterministic test suite | Confirmed | AGENTS.md; docs/test-notes.md |
| 2025-11-14 | Auth | Use Auth0 for admin authentication with roadmap to add Azure AD SSO when enterprise customers require tenant-level control | Auth0 balances fast setup with enterprise-grade features while keeping future Azure integration open | Confirmed | docs/requirements/product-requirements.md; docs/open-questions.md |
| 2025-11-14 | Respondent UX | Lock survey submissions after final submit; edits require starting a new response | Avoids compliance issues around tampering and simplifies audit history | Confirmed | docs/requirements/product-requirements.md; docs/open-questions.md |
| 2025-11-14 | Localization | Launch with English-only UI and survey templates; document translation backlog for future release | Keeps initial scope lean while acknowledging future localization demand | Confirmed | docs/requirements/product-requirements.md; docs/open-questions.md |
| 2025-11-14 | Data Residency | No regulatory storage constraints today; host in primary region with option to expand later | Simplifies infra while legal reviews continue | Confirmed | docs/requirements/product-requirements.md; docs/open-questions.md |
| 2025-11-14 | Data Retention | Default retention set to 30 days with admin-configurable overrides surfaced through support workflow | Meets enterprise expectation for configurable retention without building self-serve UI yet | Confirmed | docs/requirements/product-requirements.md; docs/open-questions.md |
| 2025-11-14 | Usage Quotas | Defer quota enforcement for workspaces/surveys in MVP | Keeps billing simple until usage patterns justify automation | Confirmed | docs/requirements/product-requirements.md; docs/open-questions.md |
| 2025-11-14 | Backend Models | Standardize on SQLAlchemy 2.0 ORM with Alembic migrations and Pydantic v2 serialization helpers | SQLAlchemy is widely supported, integrates with FastAPI, and eases onboarding/maintenance compared to the younger SQLModel | Confirmed | docs/architecture/system-architecture.md; docs/open-questions.md |
| 2025-11-14 | Frontend State | Adopt Redux Toolkit (with RTK Query for API cache) for survey builder state/offline caching | RTK provides predictable state, battle-tested tooling, and strong TS support for complex builder interactions | Confirmed | docs/architecture/system-architecture.md; docs/open-questions.md |
| 2025-11-14 | Frontend Runtime | Standardize on Next.js (App Router) with Vite for isolated library builds where needed | Next.js brings SSR/ISR out of the box, strong routing + data-fetching story, and aligns with hiring pool familiarity | Confirmed | README.md; docs/architecture/system-architecture.md |
| 2025-11-14 | Hosting & Secrets | Package services into Docker containers orchestrated via Docker Compose; store secrets in `.env.local` during dev and vault-based manager in prod | Containerization matches Raspberry Pi deployment target while keeping path open for managed hosts | Confirmed | docs/architecture/system-architecture.md; docs/open-questions.md |
| 2025-11-14 | Notifications & Uploads | Skip transactional email vendor integration and disable file uploads for initial releases | Emails handled manually today and uploads out of scope, reducing surface area | Confirmed | docs/architecture/system-architecture.md; docs/open-questions.md |

## Outstanding Decisions

| Area | Open Question | Next Steps / Owners | References |
| --- | --- | --- | --- |

> Update this log whenever a decision is made or an open question is resolved. Include links to relevant PRs, ADRs, or meeting notes for future reference.
