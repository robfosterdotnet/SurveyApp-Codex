# Testing Scaffold

Mirror runtime folders within `tests/` to keep specs discoverable. Recommended stacks:
- **Frontend** – Vitest + React Testing Library under `tests/frontend`.
- **Backend** – Pytest with httpx TestClient under `tests/backend`.

Use naming convention `*.spec.ts` for frontend and `test_*.py` for backend. Document scenarios that require MSW or other fixtures in `docs/test-notes.md` when flakiness appears.
