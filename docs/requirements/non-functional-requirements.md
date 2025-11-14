# Non-Functional & Platform Requirements – SurveyApp

This document supplements `product-requirements.md` by detailing the operational guarantees, technical constraints, and cross-cutting policies that apply to all personas.

## Quality Attributes

### Availability & Reliability
- 99.5% SLA for public respondent endpoints.
- Automated health probes, multi-AZ deployment, and graceful degradation when dependencies (Redis, Celery, DB) degrade.
- Autosave must survive transient network failures; offline drafts sync once reconnecting.

### Scalability
- Target: 1k concurrent respondents, 100 active surveys, 1M total responses.
- Back-pressure new invitation generation when concurrency exceeds 2k.
- WebSocket dashboards sustain 5 concurrent admin viewers without throttling.

### Performance
- Builder UI interactions ≤200 ms P95.
- Respondent submission end-to-end ≤1s P95.
- Autosave confirmation ≤2s from user action.
- WebSocket dashboard latency ≤3s between event and display update.

### Security & Compliance
- Encryption at rest for all data stores; keys rotated every 90 days.
- Audit trail capturing who/what/when/IP for all admin/designer/support actions, including impersonation events.
- Device + IP logged on respondent verification.
- Auth0 provides MFA for admins; Azure AD integration planned post-MVP.
- GDPR-friendly storage with anonymized exports upon request.

### Accessibility
- WCAG 2.1 AA compliance for designer/admin/ respondent flows.
- Automated accessibility audits run nightly; issues logged in `docs/test-notes.md`.
- Provide reduced-motion option and high-contrast focus indicators.

### Observability
- OpenTelemetry tracing across frontend/backend.
- Structured logs with correlation IDs.
- Metrics: completion rate, dropout per section, autosave failures, anomaly count, export duration.
- Alerts for low completion (<60%) and autosave error bursts.

## Platform & Integration Constraints
- **Frontend:** Next.js (App Router) with shadcn/ui, Redux Toolkit + RTK Query; prototypes/storyboards mirror persona journeys.
- **Backend:** FastAPI + SQLAlchemy 2.0 + Alembic; REST + WebSocket via API gateway with rate limiting.
- **Workers:** Celery + Redis handling reminders, exports, anomaly detection.
- **Database:** Postgres 15 transactional store; analytics snapshots replicated to Postgres warehouse; schema designed for future Snowflake/BigQuery migration.
- **Distribution:** Manual email delivery for MVP; system exports CSV kits (recipient + link) but does not call ESP APIs yet.
- **Hosting:** Docker/Compose for services locally and on Raspberry Pi 5 demo units; secrets via `.env.local` in dev and vault-based manager in prod.
- **State Handling:** RTK Query caches client state; server caches layered via Redis when needed.

## Data & Policy Notes
- **Localization:** English-only UI at launch; locale fields stored for future translations.
- **Respondent edits:** Not permitted post-submit; new submission required for changes.
- **Retention:** 30-day default for partial responses after close; overrides (60/90 days) require admin approval and ticket reference.
- **Data residency:** Single region for MVP; architecture must allow expansion to additional regions when legal mandates appear.
- **Quotas:** Workspace/survey quotas deferred; collect telemetry to inform future enforcement.
- **File uploads:** Deferred; builder hides the file upload type until antivirus scanning lands.

## Testing & Tooling Expectations
- Vitest + React Testing Library for frontend; Pytest for backend.
- MSW for API mocking; log flaky tests in `docs/test-notes.md`.
- Coverage goal: ≥80% lines.
- `npm run lint` integrates ESLint + Prettier + accessibility rules.

## References
- Product requirements: `docs/requirements/product-requirements.md`
- Architecture decisions: `docs/decision-log.md`
- Test notes: `docs/test-notes.md`
