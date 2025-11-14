# Stakeholder Sync – 2025-11-14

**Facilitator:** Rob (Product)  
**Participants:** Priya (Product), Max (Engineering Lead), Jordan (Frontend), Sam (Backend), Lina (DevOps), Cara (Compliance), Taylor (Support)

## Objectives

1. Walk through outstanding requirements and confirm assumptions captured in `docs/open-questions.md`.
2. Validate architecture/runtime decisions logged in `docs/decision-log.md`.
3. Capture next steps for documentation and onboarding.

## Discussion & Outcomes

### Product & Compliance
- Confirmed Auth0 as the initial IdP with Azure AD on the roadmap; Compliance is satisfied as long as tenant isolation is documented before enterprise onboarding.
- Respondent submissions remain immutable once sent; compliance emphasized importance of a clear audit log and messaging in respondent UI.
- Launching in English keeps scope tight; Product to collect localization demand post-MVP.
- Legal confirmed no immediate residency constraints. We can proceed with single-region hosting as long as encryption at rest is enabled.
- Retention defaults to 30 days with support-managed overrides; Support requested a lightweight SOP for manual adjustments.
- Usage quotas postponed until we have billing instrumentation; Finance will revisit once pilot usage rolls in.

### Architecture & Engineering
- Backend will standardize on FastAPI + SQLAlchemy 2.0 + Alembic. Engineering lead highlighted broader hiring pool familiarity and stronger migration workflows than SQLModel.
- Frontend committed to Next.js (App Router) with Redux Toolkit + RTK Query to handle builder state, draft autosave, and offline caching.
- Observability stack (OpenTelemetry, structured logs, metrics) re-affirmed as part of initial release checklist.

### Infrastructure & Operations
- Hosting target is Docker/Compose for both services. Lina will maintain Raspberry Pi 5 installation guide (Docker CE arm64 + Compose plugin) so local demos mirror production images.
- Secrets for dev live in `.env.local`; production secrets will ride in the existing vault service with Docker secrets wiring.
- Since transactional emails and uploads are out of scope, we can skip integrations with providers until the growth plan requires it.

## Decisions Captured

| Area | Decision | Reference |
| --- | --- | --- |
| Auth | Auth0-first approach, Azure AD planned | docs/open-questions.md |
| Respondent UX | Submissions locked after final submit | docs/open-questions.md |
| Localization | English-only launch | docs/open-questions.md |
| Data Residency | Single-region hosting acceptable today | docs/open-questions.md |
| Data Retention | 30-day default, support-managed overrides | docs/open-questions.md |
| Quotas | Defer enforcement until post-MVP | docs/open-questions.md |
| Backend Models | SQLAlchemy 2.0 + Alembic | docs/decision-log.md |
| Frontend Runtime & State | Next.js + Redux Toolkit/RTK Query | docs/decision-log.md |
| Hosting & Secrets | Docker/Compose, env files locally, vault in prod | docs/decision-log.md |
| Notifications/Uploads | Manual emails, file uploads out of scope | docs/open-questions.md |

## Action Items

1. Priya to add localization backlog section to `docs/requirements/product-requirements.md`.
2. Taylor to draft data-retention override SOP for support queue.
3. Sam and Jordan to author ADRs summarizing SQLAlchemy + Next.js selections for future onboarding.
4. Lina to publish the Raspberry Pi Docker setup within `/docs/architecture/`.
5. Product/Engineering to update onboarding deck with meeting summary for new team members.
