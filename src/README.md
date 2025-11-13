# Source Layout

```
src/
├── frontend/   # React + shadcn/ui client
└── backend/    # FastAPI service + workers
```

Use TypeScript + Vite/Next conventions inside `frontend`, and Python 3.11+ with FastAPI inside `backend`. Keep shared contracts (DTOs, question schemas) in `shared-contracts/` subfolders when they need to be consumed by both stacks.
