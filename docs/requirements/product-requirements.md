# Product Requirements – SurveyApp

## Vision
Provide a unified workspace where researchers can design surveys with multiple question types, distribute unique response links, and monitor completion progress plus aggregated insights.

## Personas
1. **Survey Designer** – creates surveys, defines logic, schedules launches.
2. **Respondent** – receives invitation links and submits answers.
3. **Administrator** – supervises survey programs, reviews analytics, manages permissions.

## Functional Requirements
- Create, edit, duplicate, archive surveys with metadata (title, description, category, closing date, owner).
- Manage dynamic sections with branching logic and reusable question banks.
- Support question types: single choice, multiple choice, Likert scale, ranking, numeric input, free text, date, matrix/grid, file upload.
- Generate unique, expiring response links per recipient with optional authentication (email token or SSO placeholder).
- Capture partial responses and resume sessions before closing date.
- Provide admin dashboard summarizing total responses, completion rate, and outlier alerts.
- Export data to CSV and schedule snapshots to Postgres-backed warehouse tables.

## Non-Functional Requirements
- Target SLA: 99.5% availability for public survey endpoints.
- Initial scale: 1k concurrent respondents, 100 surveys, 1M total responses.
- GDPR-friendly data storage with encryption at rest and audit trail of admin actions.
- Accessibility: WCAG 2.1 AA for respondent experience and admin UI.

## Integration Requirements
- React 18 + shadcn/ui front-end served over Vite/Next (to be finalized in architecture doc).
- FastAPI backend exposing REST + WebSocket (for live stats) behind API gateway.
- Postgres 15 for transactional data; future analytics warehouse TBD.
- Email delivery provider (placeholder) for sending invitation links.

## Open Questions
- What authentication/authorization provider will admins use?
- Should respondents be able to edit submissions after completion?
- Do surveys need translation/localization at launch?
- Are there regulatory constraints for data residency?
