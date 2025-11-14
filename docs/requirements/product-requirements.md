# Product Requirements – SurveyApp

## Vision
Provide a unified workspace where researchers can craft mid-complexity surveys (<30 questions and ≤3 respondent segments), distribute secure invitations, and surface insights fast enough to intervene before a survey underperforms. The MVP must enable:
- Survey Designers to launch a branching survey with rich question types in <30 minutes.
- Administrators to detect an at-risk survey (dropout spike, low completion) within 15 minutes of anomaly onset.
- Respondents to complete a branded, WCAG-compliant experience on mobile or desktop with trust-building cues.

> Non-functional goals (availability, performance, security, observability, platform constraints) now live in `docs/requirements/non-functional-requirements.md`.

## Experience Principles
1. **Guided** – Every persona sees contextual checklists, status indicators, and tooltips so complex steps feel approachable.
2. **Trustworthy** – Autosave, privacy disclosures, and immutable audit trails reinforce that data is safe and accurate.
3. **Human & Warm** – Friendly language, modern gradients, and micro-animations keep flows inviting even when tasks are operational.
4. **Deterministic** – Systems expose state clearly (link status, autosave timestamps, launch readiness) to avoid surprises.

## Personas
### Survey Designer (e.g., Sofia – Research Lead)
- **Goals:** Build reusable survey templates, configure branching logic, and coordinate launches with marketing/sales partners.
- **Success metrics:** Draft-to-launch time under 30 minutes, zero validation errors at publish, collaborator visibility.
- **Constraints:** Needs autosave, reusable question banks, timezone-aware scheduling, and ability to export recipient kits for manual emails.

### Respondent (e.g., Alex – Customer Success Manager)
- **Goals:** Complete assigned surveys quickly from any device, understand how data is used, and trust that submissions are final.
- **Success metrics:** Completion rate ≥80%, ability to pause/resume within 30 days, no accessibility blockers.
- **Constraints:** Receives manual email with expiring magic link, English-only UI at launch, cannot edit responses post-submit.

### Administrator (e.g., Morgan – Program Admin)
- **Goals:** Monitor health metrics, manage permissions, enforce retention policies, and coordinate recipient distribution.
- **Success metrics:** At-risk surveys flagged within 15 minutes, retention overrides logged, CSV exports delivered daily, zero unauthorized access.
- **Constraints:** Auth0 for admin SSO (Azure AD roadmap), Docker/Compose deployment, manual email distribution, no quotas in MVP.

## Persona Journeys (Happy Paths)
1. **Designer Journey**
   1. Kick off new survey via template or duplicate; fill metadata (title, description, category, closing date, owners, objective tags).
   2. Arrange sections, configure branching using respondent attributes or answer thresholds, and pull in question bank entries (Likert, ranking, numeric, text, date, matrix).
   3. Preview desktop/mobile plus screen-reader narration; resolve WCAG cues and autosave confirmations.
   4. Schedule timezone-aware windows, upload CSV recipients, generate expiring links for manual send, and secure admin approval to publish.
2. **Respondent Journey**
   1. Receive manual invitation email with magic link; verify token (device + IP logged).
   2. Land on welcome card detailing estimated time, sections, privacy + 30-day partial retention, and ability to resume via same device.
   3. Complete sections with autosave every 10 seconds, accessible controls, and inline help; optional reminder email nudges if paused.
   4. Review answers (pre-submit only) and confirm final submission; system locks edits, displays confirmation ID, and offers receipt email.
3. **Administrator Journey**
   1. Morning scan of dashboard (completion, dropout, anomalies, median time) refreshing via WebSocket every 30 seconds.
   2. Manage distribution: validate CSV columns, generate expiring links, export send kit for marketing, revoke/resend with throttle guards.
   3. Enforce policies: approve retention overrides (default 30 days), monitor audit logs, manage role access & impersonation requests.
   4. Export CSV + hourly snapshots, push to warehouse, and share digest (Slack/email) with stakeholders.

## Functional Requirements

### 1. Designer Workspace
| Capability | Description | Acceptance Criteria |
| --- | --- | --- |
| Metadata studio | Capture title, description (Markdown), category, owner(s), business objective tags, closing date, retention notes. | All fields autosave within 10s; metadata surfaces in admin dashboards and exports. |
| Section + question builder | Drag/drop sections; configure branching from previous answers, scores, or recipient attributes. Supported question types: single select, multi select, Likert, ranking, numeric, free text, date, matrix. File uploads deferred. | Builder prevents orphaned branches; each question exposes validation (min/max, regex, required, help text). |
| Question bank | Maintain reusable templates; inserting a template copies defaults while tracking provenance for future updates. | Designers can search/filter bank, preview question, and update local copy without mutating the bank version. |
| Preview & quality | Dual-pane preview (desktop/mobile) plus screen reader transcript; indicates autosave timestamp, translation fallback (English), and warnings (missing help text, accessibility issues). | WCAG AA checklist passes before publish; simulated personas (segment, locale) follow the correct branch path. |
| Launch readiness | Checklist verifying metadata completeness, sections validated, preview approved, recipients uploaded, and admin approval recorded. | Publish button disabled until checklist passes; scheduling supports start/end windows with timezone selection and optional reminders. |
| Distribution handoff | Upload CSV (`email`, `name`, `segment`, `locale`), generate expiring links, export kit (CSV + optional PDF summary) for manual email blast. | System logs generation timestamp, user, and number of links; kit includes retention + privacy boilerplate. |

### 2. Respondent Experience
| Capability | Description | Acceptance Criteria |
| --- | --- | --- |
| Invitation & verification | Manual email contains CTA with expiring magic link (48–72h configurable). Clicking validates token, device, IP, and status before entry. | Expired/invalid links gracefully inform respondent and direct them to support; tokens are single-use until completion. |
| Welcome & consent | Warm welcome screen shows estimated duration, section count, privacy statement (30-day draft retention), contact info, and timezone confirmation. | English copy only at launch but architecture stores locale for future translations; respondents must acknowledge privacy before continuing. |
| Form interaction | Cards highlight current section with progress percent, show inline help, and support keyboard/touch navigation. Autosave occurs every 10 seconds and on section navigation; indicator confirms success. | Respondents can resume on same device without re-auth; cross-device resume requires token revalidation per security policy. |
| Pause & reminder | Respondents may exit mid-survey; partial responses retained 30 days beyond closing date and optional reminder email sent (manual for MVP). | Reminder cadence configurable per survey; audit log records reminder creation. |
| Submission & receipt | Review page lists sections with edit links (pre-submit). Submissions are immutable; final screen confirms lock, displays confirmation ID, and lets respondent email themselves a receipt. | Post-submit link displays summary-only view; any modification request directs respondent to admins (no edits). |

### 3. Administrator & Program Management
| Capability | Description | Acceptance Criteria |
| --- | --- | --- |
| Auth & roles | Admin login handled via Auth0 (enterprise plan). Roles: Admin, Designer, Support (impersonation). Azure AD SSO on roadmap. | MFA enforced; impersonation actions require justification and time out after 15 minutes, logging who/what/when/IP. |
| Operations dashboard | WebSocket-powered cards show completion rate, dropout hotspots, median completion time, anomalies (z-score >3), open incidents, and SLA timers. | Refresh every 30s; anomalies trigger toast + assignable task to designer. |
| Distribution console | Validate CSV uploads, track link status (queued/sent/bounced/revoked), throttle resends, trigger revocations, and export manual-send kit. | Per-recipient log with status history; revocation instantly invalidates token. |
| Governance & retention | Default retention 30 days; support-managed overrides allow 60/90-day policies. Localization English-only; data residency single-region until legal requires expansion. | Override workflow captures ticket ID, reason, expiration; system enforces purge schedule and surfaces upcoming purges. |
| Support tools | Support persona can impersonate designer (no respondent PII), reissue links, and annotate cases. | All actions recorded; impersonation UI highlights read-only vs editable zones. |
| Analytics & exports | Hourly snapshots feed Postgres warehouse tables; admins export CSV filtered by segment, date, owner. | Exports respect RBAC; jobs visible with retry/alerts; CSV includes metadata, timestamps, respondent IDs. |

## Out-of-Scope / Future Enhancements
- Automated email delivery + webhook reconciliation (manual for MVP).
- File upload question type + antivirus scanning.
- Localization beyond English.
- Automated quota enforcement per workspace/survey.
- Native mobile apps (responsive web only now).

## Decision Traceability
| Topic | Decision | Reference |
| --- | --- | --- |
| Auth provider | Auth0 now, Azure AD later | docs/decision-log.md |
| Respondent edits | Submissions locked post-submit | docs/decision-log.md |
| Localization | English-only launch | docs/decision-log.md |
| Data residency | Single-region hosting acceptable now | docs/decision-log.md |
| Retention | 30-day default with admin override | docs/decision-log.md |
| Quotas | Deferred for MVP | docs/decision-log.md |
| Question builder ORM | SQLAlchemy 2.0 + Alembic | docs/decision-log.md |
| Frontend runtime/state | Next.js + Redux Toolkit/RTK Query | docs/decision-log.md |
| Hosting & secrets | Docker/Compose, env files locally, vault prod | docs/decision-log.md |
| Notifications/uploads | Manual emails, no file uploads | docs/decision-log.md |

All open questions have been resolved; this document is ready for design sign-off before implementation.
