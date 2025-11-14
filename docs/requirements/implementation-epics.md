# Implementation Epics – SurveyApp

Derived from `product-requirements.md` and `non-functional-requirements.md`. Each epic includes objective, scope, completion criteria, dependencies, and key deliverables.

---

## Epic 1 – Designer Workspace Foundation
- **Objective:** Enable Survey Designers to create, edit, and autosave survey metadata, sections, and question content within 30 minutes per survey.
- **Scope:**
  - Metadata studio (title, description, category, owners, objective tags, closing date, retention notes).
  - Drag-and-drop section builder with autosave + validation.
  - Question palette covering required types (single/multi select, Likert, ranking, numeric, text, date, matrix) plus inline validation settings.
  - Question bank CRUD with template insertion and provenance tracking.
- **Done when:**
  - Autosave persists every 10 seconds and on navigation; offline drafts resync after reconnecting.
  - Builder prevents orphaned branches and missing validation rules.
  - Metadata surfaces in admin dashboards/exports.
- **Dependencies:** Redux Toolkit state layer, SQLAlchemy models/migrations for surveys/questions, shadcn/ui component primitives.
- **Deliverables:** Storybook/Next.js prototype pages, API endpoints for surveys/sections/questions, automated tests (Vitest + Pytest).

## Epic 2 – Branching Logic & Preview Quality
- **Objective:** Provide designers with confidence that branching paths and accessibility requirements behave correctly before launch.
- **Scope:**
  - Natural-language branching editor referencing respondent attributes and answer thresholds.
  - Dual-pane preview (desktop + mobile) plus screen-reader transcript toggle.
  - Persona simulator (happy path, churn risk) with autosave + retention cues.
  - Launch readiness checklist gating publish.
- **Done when:**
  - Branch graph validation reports zero dead ends for each segment.
  - WCAG AA automation passes before enabling publish.
  - Preview shows autosave timestamps and locale fallback (English).
- **Dependencies:** Epic 1 UI foundation, OpenTelemetry hooks for preview metrics, accessibility tooling (axe/Vitest).
- **Deliverables:** Preview route, branching validation service, checklist UI + API, regression tests.

## Epic 3 – Respondent Experience & Autosave
- **Objective:** Deliver a warm, trustworthy respondent journey with secure invitations, autosave/resume, WCAG compliance, and immutable submissions.
- **Scope:**
  - Magic-link verification flow (token validity, device/IP logging, graceful expiry errors).
  - Welcome/consent screen with estimated duration, privacy, retention (30-day partial storage).
  - Sectioned form with autosave indicator, accessible controls, inline help, and reduced-motion option.
  - Pause/reminder handling (manual reminder trigger, 30-day retention), submission receipt + confirmation ID.
- **Done when:**
  - P95 submit latency ≤1s; autosave confirmation ≤2s.
  - Resume on same device restores state without re-auth; cross-device requires token revalidation.
  - Post-submit link becomes read-only summary; edits require support path.
- **Dependencies:** Token service, Redis/Celery for reminder queueing, email template for manual reminder exports.
- **Deliverables:** Respondent Next.js page, FastAPI endpoints for autosave/submission, MSW-backed tests, accessibility audit scripts.

## Epic 4 – Distribution & Link Management
- **Objective:** Allow admins to upload recipient CSVs, generate unique expiring links, export manual-send kits, and track delivery status.
- **Scope:**
  - CSV validation (email, name, segment, locale columns).
  - Link generation service with expiration, revocation, resends with throttle guardrails.
  - Distribution console showing status per recipient (queued/sent/bounced/revoked).
  - Export kit (CSV + optional PDF summary) for manual email campaigns.
- **Done when:**
  - Audit log records link creation/revocation with user + timestamp.
  - Resend throttle prevents >1 resend per 10 minutes per recipient.
  - Export kit includes retention/privacy boilerplate and scheduling summary.
- **Dependencies:** Auth0 role enforcement, Postgres tables for recipients/tokens, Celery for status jobs.
- **Deliverables:** Admin UI console, backend services, CSV parser utilities, automated tests verifying revocation/resend logic.

## Epic 5 – Admin Dashboard, Governance, & Analytics
- **Objective:** Give administrators live visibility into survey health, retention policies, access controls, and exports.
- **Scope:**
  - WebSocket-powered dashboard (completion rate, dropout per section, median time, anomalies z-score >3).
  - Retention override workflow (default 30 days, support-managed 60/90 days) with ticket references.
  - Support/impersonation tooling with 15-minute timeout and justification logging.
  - CSV export + hourly snapshot jobs with status monitoring.
- **Done when:**
  - Dashboard refreshes every 30s and alerts route to designers.
  - Overrides update purge schedule and display upcoming purges.
  - Exports respect RBAC and produce Postgres-ready CSVs with metadata + respondent IDs.
- **Dependencies:** OpenTelemetry + metrics, Celery snapshot pipeline, Auth0 role claims.
- **Deliverables:** Admin dashboard UI, FastAPI WebSocket/feed endpoints, retention API, export job orchestration, monitoring dashboards.

## Epic 6 – Platform, Observability, and Dev Experience
- **Objective:** Establish the infrastructure, tooling, and non-functional foundations required by both functional epics.
- **Scope:**
  - Docker/Compose setup (frontend, backend, workers, Postgres, Redis) with Raspberry Pi 5 instructions.
  - Environment management (`.env.local`, vault integration) and secrets handling.
  - OpenTelemetry instrumentation, structured logging, metrics/alerts.
  - CI scripts for lint/test/coverage, plus MSW/Vitest/Pytest baselines.
- **Done when:**
  - `npm run dev`, `npm run build`, `npm run start`, and backend equivalents run via Compose.
  - Telemetry dashboards show key metrics (completion, autosave errors, anomaly counts).
  - CI enforces ≥80% coverage and ESLint/Prettier compliance.
- **Dependencies:** None (foundational), but blocks all feature epics.
- **Deliverables:** Docker configs, CI pipeline, documentation (`docs/architecture`, `docs/test-notes.md` updates), monitoring setup.
