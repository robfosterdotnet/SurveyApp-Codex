# Implementation Epics – SurveyApp

Derived from `product-requirements.md` and `non-functional-requirements.md`. Each epic includes objective, scope, completion criteria, dependencies, and key deliverables.

---

## Epic 0 – Environment & Platform Foundations
- **Objective:** Provision Docker/Compose scaffolding, telemetry, and CI tooling so feature teams can build with confidence.
- **Scope:**
  - Docker/Compose setup (frontend, backend, workers, Postgres, Redis) with Raspberry Pi 5 instructions.
  - Environment management (`.env.local`, vault integration) and secrets handling.
  - OpenTelemetry instrumentation, structured logging, metrics/alerts.
  - CI scripts for lint/test/coverage, plus MSW/Vitest/Pytest baselines.
- **Done when:**
  - `npm run dev`, `npm run build`, `npm run start`, and backend equivalents run via Compose.
  - Telemetry dashboards show key metrics (completion, autosave errors, anomaly counts).
  - CI enforces ≥80% coverage and ESLint/Prettier compliance.
- **Dependencies:** None (foundational), blocks all other epics.
- **Deliverables:** Docker configs, CI pipeline, documentation updates, monitoring setup.

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

### Key Features

1. **Designer Shell Setup**
   - Build survey metadata form (title, description, category, owner, tags, closing date, retention notes).
   - Hook autosave service that stores changes every 10 seconds and on field blur.
   - Acceptance: metadata visible in API responses and admin dashboard cards.
2. **Section Builder**
   - Drag/drop sections with ordering, duplication, and delete safeguards.
   - Define section-level branching placeholders (to be wired fully in Epic 2).
   - Acceptance: sections persisted, reordering reflected in payloads, validation prevents empty titles.
3. **Question Palette**
   - UI to add supported question types with validation config (required, min/max, regex, help text).
   - Integrate with Redux Toolkit slice and backend models.
   - Acceptance: each type renders preview stub; validations saved and reloaded.
4. **Question Bank CRUD**
   - Bank listing with search/filter, create/edit forms, insert into survey copy with provenance reference.
   - Acceptance: inserting bank question duplicates config yet retains reference ID for sync audits.

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

### Key Features

1. **Natural-Language Rule Builder**
   - Parser for conditional statements referencing question answers or respondent attributes.
   - UI chips to compose “If [question] [operator] [value] then [section/question]”.
   - Acceptance: rules serialize to backend schema and evaluate in preview simulator.
2. **Branch Graph Validator**
   - Service that walks all sections per segment to detect dead ends or loops.
   - Provide UI error panel with actionable fixes.
   - Acceptance: failing surveys display blocking errors; all paths green enable publish.
3. **Dual-Pane Preview**
   - Render desktop + mobile simultaneously, share form state, show autosave indicator and timestamps.
   - Add screen-reader transcript overlay for accessibility review.
   - Acceptance: preview respects branching rules, shows English fallback when locales missing.
4. **Launch Readiness Checklist**
   - Checklist component tied to metadata, preview, distribution readiness, approvals.
   - Backend flags gating publish endpoint until checklist satisfied.
   - Acceptance: incomplete tasks disable publish/Schedule; completion logs timestamp + user.

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

### Key Features

1. **Invitation & Verification Flow**
   - Magic link landing route verifying token status, device, IP; handle expired/invalid cases with support CTA.
   - Backend endpoint to mint/verify tokens with single-use enforcement.
   - Acceptance: valid token proceeds, invalid shows friendly error; logs audit trail.
2. **Welcome & Consent Screen**
   - Display estimated duration, sections, privacy/retention details, timezone confirmation, contact info.
   - Require acknowledgment before entering survey.
   - Acceptance: consent recorded with timestamp and stored alongside response metadata.
3. **Sectioned Form + Autosave**
   - Implement progress rail, autosave indicator (every 10s + on navigation), inline help, reduced-motion toggle.
   - Autosave API supports optimistic updates/local fallback.
   - Acceptance: reload restores progress; accessibility checks pass (focus order, ARIA).
4. **Submission & Receipt**
   - Review page with edit buttons before submit; final confirmation locks responses and shows confirmation ID + email receipt option.
   - Post-submit route read-only, surfaces support guidance.
   - Acceptance: new submissions immutable, receipt includes retention policy text.

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

### Key Features

1. **Recipient CSV Importer**
   - Upload flow with validation (headers, email format, locale codes) and preview table.
   - Store recipients per survey with import metadata.
   - Acceptance: invalid rows flagged inline; successful import logged.
2. **Link Generation Service**
   - Generate unique expiring tokens per recipient; configure expiration window, per-segment overrides.
   - Provide API to revoke/extend links.
   - Acceptance: tokens stored with status; revocation invalidates immediately.
3. **Distribution Console UI**
   - Table showing status (queued/sent/bounced/revoked) with filters, search, and resend controls honoring throttle.
   - Visual indicators for segments/locales.
   - Acceptance: status updates reflect backend state; throttle prevents rapid resends.
4. **Manual Send Kit Export**
   - Export CSV (and optional PDF summary) including recipient, link, segment, schedule, privacy boilerplate.
   - Track who exported and when.
   - Acceptance: kit download completes under 5s for 5k recipients; audit log entry created.

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

### Key Features

1. **Live Health Dashboard**
   - WebSocket feed powering cards for completion rate, dropout, median time, anomalies, SLA timer.
   - Filtering by survey, segment, locale.
   - Acceptance: metrics update every 30s; anomalies highlight >3σ deviations.
2. **Retention Override Workflow**
   - UI modal to request/approve overrides (60/90 days) with ticket ID and expiry date.
   - Backend scheduler to enforce purge deadlines.
   - Acceptance: overrides visible in policy table; upcoming purges surfaced in dashboard.
3. **Support/Impersonation Console**
   - Allow Support role to impersonate designers for troubleshooting without exposing respondent PII.
   - Enforce 15-minute timeout and justification entry.
   - Acceptance: impersonation actions logged; UI clearly indicates impersonation mode.
4. **Export & Snapshot Jobs**
   - Trigger CSV exports filtered by segment/date/owner; monitor hourly snapshot jobs pushing to warehouse.
   - Provide retry controls and job status history.
   - Acceptance: exports respect RBAC; failed jobs send alerts and can be retried from UI.
