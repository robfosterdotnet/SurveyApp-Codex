# SurveyApp Documentation Hub

This directory houses the planning assets for SurveyApp-Codex. Keep stakeholder-facing summaries near the top level and detailed specs in subfolders:

- `requirements/` – Business goals, personas, and feature-level acceptance criteria.
- `architecture/` – System diagrams, technology decisions, and deployment notes.
- `user-stories.md` – Backlog sliced by persona with INVEST-friendly acceptance.

When a document grows larger than ~200 lines, consider breaking it into a dedicated file and link it from here. Reference shared definitions (question types, statuses, etc.) instead of duplicating them so implementation stays consistent across the frontend (React + shadcn/ui) and backend (FastAPI + Postgres).
