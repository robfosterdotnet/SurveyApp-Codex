# SurveyApp-Codex

Planning workspace for a survey design and analytics platform powered by React (shadcn/ui), FastAPI, and Postgres.

## Repository Map

- `docs/` – Requirements, architecture notes, and user stories to guide implementation.
- `src/frontend/` – React client scaffold with folders for components, features, services, lib helpers, and shared types.
- `src/backend/` – FastAPI scaffold (api, core, models, schemas, services, workers) for the server + background jobs.
- `tests/` – Mirrors runtime folders for Vitest/RTL and Pytest suites.

## Next Steps

1. Finalize requirements inside `docs/requirements/` and expand user stories with acceptance criteria.
2. Decide on the React runtime (Vite vs. Next.js) and bootstrap the app within `src/frontend`.
3. Stand up the FastAPI project under `src/backend` with database migrations and env config.
