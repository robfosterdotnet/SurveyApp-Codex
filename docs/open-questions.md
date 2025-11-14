# Open Questions Log

| ID | Question | Source Doc | Notes / Next Step | Owner |
|----|----------|------------|-------------------|-------|
| Q1 | What authentication/authorization provider will admins use? | `docs/requirements/product-requirements.md` | **Resolved (2025-11-14):** Start with Auth0 for admin auth, keep Azure AD integration on the roadmap for enterprise SSO parity. | Product/Infra |
| Q2 | Should respondents be able to edit submissions after completion? | `docs/requirements/product-requirements.md` | **Resolved (2025-11-14):** Lock responses after final submission; edits require opening a new response to preserve audit history. | Product |
| Q3 | Do surveys need translation/localization at launch? | `docs/requirements/product-requirements.md` | **Resolved (2025-11-14):** Launch with English-only UI+templates; track localization backlog for later release once demand proven. | Product |
| Q4 | Are there regulatory constraints for data residency? | `docs/requirements/product-requirements.md` | **Resolved (2025-11-14):** No residency mandate now; default to primary region hosting and revisit when legal guidance changes. | Legal/Infra |
| Q5 | How will enterprise customers request custom data retention policies? | `docs/requirements/product-requirements.md` | **Resolved (2025-11-14):** Default 30-day retention; allow overrides through support workflow until self-serve tooling exists. | Product/Support |
| Q6 | Do we need built-in quota enforcement per workspace or per survey? | `docs/requirements/product-requirements.md` | **Resolved (2025-11-14):** Defer quota enforcement for MVP to simplify billing; collect usage metrics to revisit. | Product/Finance |
| Q7 | Pick between SQLModel vs. SQLAlchemy ORM for FastAPI models. | `docs/architecture/system-architecture.md` | **Resolved (2025-11-14):** Adopt SQLAlchemy 2.0 ORM with Alembic migrations and Pydantic v2 serializers for maturity and tooling depth. | Backend |
| Q8 | Choose state management (Redux Toolkit vs. Zustand) for builder interactions/offline caching. | `docs/architecture/system-architecture.md` | **Resolved (2025-11-14):** Use Redux Toolkit + RTK Query to manage complex builder state and autosave/offline cache. | Frontend |
| Q9 | Determine hosting (managed container service vs. Kubernetes) and secrets manager. | `docs/architecture/system-architecture.md` | **Resolved (2025-11-14):** Package services via Docker/Compose; for Raspberry Pi 5 run Docker CE (arm64) with Compose plugin, store secrets in `.env.local` for dev and vault-backed manager in prod. | Infra |
| Q10 | Select email provider (Resend, Postmark, SES) and virus scanning approach for file uploads. | `docs/architecture/system-architecture.md` | **Resolved (2025-11-14):** No transactional email vendor or upload scanning needed now—emails sent manually and file uploads out of scope. | Product/Infra |

# Answers: 
 -Q1: Let's just do Auth0 for now but plan to also integrate with Azure in the future.
 
 -Q2: No, once a survey is submitted it should not be able to be edited.

 -Q3: No, English only for now.

 -Q4: No regulatory contrainsts for now.

 -Q5: Let's set an initial retention date of 30 days but make it adjustable.

 -Q6: No quotas for now.

 -Q7: Adopt SQLAlchemy 2.0 ORM with Alembic migrations.
 
 -Q8: Use Redux Toolkit + RTK Query for state/offline caching.
 
 -Q9: Deploy via Docker/Compose; install Docker CE + Compose on Raspberry Pi 5 using the Raspberry Pi OS arm64 instructions referenced above.
 
 -Q10: Manual email distribution for now; omit file uploads and virus scanning.
