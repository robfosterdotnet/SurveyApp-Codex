# Repository Guidelines

## Project Structure & Module Organization
The repository currently contains only `README.md`, so every contribution helps define our baseline. Organize new TypeScript code under `src/` with feature-focused folders (`src/features/surveys`, `src/components`, `src/lib`). Keep integration helpers in `src/services` and cross-cutting utilities in `src/shared`. Static assets belong in `public/` while architectural notes or ADRs live in `docs/`. Create a top-level `tests/` tree that mirrors the runtime folders (`tests/features/surveys`, etc.) so navigation stays predictable.

## Build, Test, and Development Commands
Use Node 20+ and npm. Once `package.json` is in place, run `npm install` to hydrate dependencies. `npm run dev` should launch the SurveyApp development server on port 3000, `npm run build` must emit the production bundle, and `npm run start` should run that bundle locally to verify environment variables. Keep `npm run lint` wired to ESLint + Prettier so formatting stays consistent and accessibility rules trigger early.

## Coding Style & Naming Conventions
Default to strict TypeScript and 2-space indentation. Prefer named exports, avoid default export barrels, and keep files under ~200 lines. Components are `PascalCase` (`SurveyEditorPanel.tsx`), hooks use `useCamelCase`, utilities are `camelCase`, and tests mirror their subjects with `.spec.ts` suffixes. Run Prettier before committing and ensure ESLint includes React, Testing Library, and accessibility plugins. Store environment-specific values in `.env.local`; never bake them into source.

## Testing Guidelines
Adopt Vitest or Jest plus React Testing Library. House suites in `tests/features/<feature>/<subject>.spec.ts` and write behavior-first assertions (“submits branching survey when all questions answered”). Aim for ≥80% line coverage using `npm test -- --coverage`. Mock remote calls with MSW to keep runs deterministic and log flaky scenarios in `docs/test-notes.md` so regressions can be replayed.

## Commit & Pull Request Guidelines
Git history currently shows only `Initial commit`, so set the tone with Conventional Commits (for example `feat(form-builder): add range question type`). Describe why the change matters, reference an issue ID, and mention migrations, seeds, or env vars in the body. Pull requests should include: concise summary, screenshots or GIFs for UI shifts, a checklist of commands run (`npm test`, `npm run lint`), and deployment considerations. Keep diffs small enough to review (~400 lines) and rebase if main advances.

## Security & Configuration Tips
Load secrets through `.env*` files and expose only safe values with prefixes such as `NEXT_PUBLIC_`. Keep respondent data scrubbed; anonymized fixtures belong under `seed/fixtures`. Enable dependency scanning (Dependabot or Renovate) once package metadata exists so survey tooling stays patched.
