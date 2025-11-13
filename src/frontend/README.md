# Frontend Scaffold

- `components/` – Reusable shadcn/ui-driven building blocks.
- `features/` – Route- or domain-specific modules (start with `surveys/`).
- `services/` – API clients, WebSocket helpers, auth adapters.
- `lib/` – Cross-cutting utilities (form hooks, validators, analytics helpers).
- `shared/` – Types and constants that should never reach backend-only code.

Recommended next steps:
1. Bootstrap the React app (e.g., Vite + React or Next.js) and point it here.
2. Install `shadcn/ui`, Tailwind, and state management of choice.
3. Mirror feature folders under `tests/frontend` with Vitest + Testing Library suites.
