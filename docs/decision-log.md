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

## Outstanding Decisions

| Area | Open Question | Next Steps / Owners | References |
| --- | --- | --- | --- |
| Auth | Which provider handles admin authentication/authorization? | Evaluate existing IdP options (Auth0, Azure AD, custom SSO) and document requirements | docs/requirements/product-requirements.md |
| Respondent UX | Can respondents edit submissions after final submit? | Determine compliance impact and UX complexity; capture in requirements | docs/requirements/product-requirements.md |
| Localization | Do surveys need translation/localization support at launch? | Interview stakeholders, scope i18n requirements for both UI and survey content | docs/requirements/product-requirements.md |
| Data Residency | Are there region-specific storage mandates? | Consult legal/compliance, decide on hosting regions and encryption policies | docs/requirements/product-requirements.md |
| Frontend Runtime | Choose between Vite vs. Next.js and settle on client state management (Redux Toolkit vs. Zustand vs. Recoil) | Prototype both approaches, record trade-offs in architecture doc | README.md; docs/architecture/system-architecture.md |
| Backend Models | Pick SQLModel vs. SQLAlchemy ORM for FastAPI | Compare developer experience vs. maturity; confirm migration strategy | docs/architecture/system-architecture.md |
| Hosting & Secrets | Decide on deployment target (managed containers vs. Kubernetes) and secrets manager | Engage DevOps to align with org standards | docs/architecture/system-architecture.md |

> Update this log whenever a decision is made or an open question is resolved. Include links to relevant PRs, ADRs, or meeting notes for future reference.
