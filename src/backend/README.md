# Backend Scaffold

```
backend/app/
├── api/        # FastAPI routers grouped by domain
├── core/       # Settings, logging, dependency injection
├── models/     # ORM models targeting Postgres
├── schemas/    # Pydantic schemas for request/response bodies
├── services/   # Business logic, integrations (email, analytics)
└── workers/    # Background tasks (Celery/RQ) for async processing
```

Add `alembic/` or `migrations/` once schema work starts. Keep environment variables documented in `docs/requirements` and load them through `core/config.py`.
