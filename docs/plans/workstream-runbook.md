# Runbook – Creating Workstreams for SurveyApp Epics

Audience: Junior developers or project coordinators who need to decompose SurveyApp requirements into actionable workstreams using the existing requirements and planning artifacts.

Outcome: A set of epic-specific plan docs (like `docs/plans/epic-0-environment-plan.md`) that enumerate workstreams, tasks, dependencies, acceptance checks, and sequencing so feature teams can execute without ambiguity.

---

## 1. Source Material Checklist

Review these files before drafting any workstream so requirements stay aligned and referenced snippets stay accurate:
- `README.md` – repository objectives and runtime layout.
- `docs/requirements/product-requirements.md` – personas, happy paths, and feature acceptance criteria.
- `docs/requirements/non-functional-requirements.md` – SLAs, security, accessibility, observability, and platform constraints you must bake into tasks.
- `docs/requirements/implementation-epics.md` – authoritative list of epics, objectives, scopes, dependencies, and done definitions.
- `docs/plans/epic-0-environment-plan.md` – canonical plan format (workstream tables + milestones) to mirror for every new epic.
- `docs/open-questions.md`, `docs/user-stories.md`, and `docs/test-notes.md` – supporting context and risks to address inside tasks where relevant.

Tip: Keep all files open side by side. Every workstream should map back to a requirement line item (functional or non-functional) so stakeholders can trace coverage.

## 2. Prep & Tooling

1. Confirm you have Node 20+, npm, Docker, and Python 3.11+ installed (see Epic 0 plan prerequisites) so you can validate any scripts you mention.
2. Decide which epic you are planning (e.g., Epic 2 – Branching Logic & Preview Quality) and create a scratchpad for notes.
3. Copy the workstream table template from Section 6 into your editor; you will reuse it for each workstream.
4. If the epic spans frontend, backend, and ops deliverables, pre-label swim lanes (UI, API, Platform, QA) to keep scope balanced.
5. Keep terminal tabs ready to test every command you prescribe—plans must specify runnable workflows, not just intentions.

## 3. Analysis Workflow for Each Epic

Follow this exact sequence per epic to avoid missing dependencies:

1. **Restate the Epic Objective:** Quote or paraphrase the "Objective" and "Done when" bullets from `implementation-epics.md`. This becomes the intro paragraph of the new plan file so readers see the mission up front.
2. **Extract Feature Lines:** Under the epic’s “Key Features,” list each feature ID (e.g., “Natural-Language Rule Builder”) alongside the acceptance bullets. This is your master backlog.
3. **Map Requirements to Personas:** Use `product-requirements.md` persona journeys to understand what success looks like for Designers, Respondents, and Administrators. Call out which persona drives each feature and whether cross-persona touchpoints exist.
4. **Overlay Non-Functional Constraints:** From `non-functional-requirements.md`, pull any SLAs, telemetry needs, accessibility targets, or compliance requirements that influence implementation. Annotate them next to each feature so they turn into explicit tasks later (e.g., WCAG AA automation for Epic 2).
5. **Identify Dependencies:** The epics list upstream requirements (e.g., Epic 1 needs Redux Toolkit state). Add any hidden dependencies you discover (APIs, migrations, Auth0 integration) so workstreams respect sequencing.
6. **Draft Workstream Candidates:** Group related features/constraints into 3–5 coherent workstreams. Examples:
   - Designer UI + Autosave shell
   - Backend services & database models
   - Telemetry & validation tooling
   - Distribution & operations enablement
   - QA + testing guardrails
7. **Validate Coverage:** Check that every scope item and acceptance bullet from the epic maps to at least one workstream. If not, create another workstream or expand an existing one’s task list.

Only after these steps should you start writing the plan doc. Every task you add later must cite the exact requirement line you derived it from.

## 4. Writing the Plan Doc

Create a new Markdown file under `docs/plans/` named `epic-<n>-<short-name>-plan.md`. Structure it like the Epic 0 plan:

1. **Header:** Title (`# Plan – Epic <n>: <Name>`), link back to `implementation-epics.md`, and restated objectives.
2. **Objectives & Success Metrics:** Bullet the objective and “Done when” language so readers know the bar.
3. **Prerequisites:** Hardware/software assumptions, platform constraints, sample data seeds, and security approvals required before workstreams start. Reference Raspberry Pi steps or CI tooling as appropriate.
4. **Workstreams Section:** For each workstream, insert a table using the template below populated with concrete tasks. Immediately underneath the table, add a “Task Details” subsection where you spell out every command, file path, code snippet, and validation step needed to deliver those tasks. Junior developers should be able to follow the doc without external coaching.
5. **Sequencing & Milestones:** Define 2–4 milestones that represent natural checkpoints (e.g., “Milestone A – Branch Graph Validator passes sample survey”). Tie each milestone to the tasks that must be complete.
6. **Risks & Mitigations:** Summarize open questions or blockers noted in `docs/open-questions.md` and describe mitigation plans.
7. **Exit Criteria:** Explicit proof points (commands, dashboards, test suites) showing the epic is shippable.

## 5. Task Detail Playbook

Each workstream needs prose instructions similar to the Epic 0 expansions. Use this checklist per task:
- **File Paths:** State exactly where new files live (`src/backend/app/core/settings.py`). If you reference an existing file, include the relative path and anchor (e.g., `docs/requirements/product-requirements.md#designer-workspace`).
- **Sample Code:** Provide fenced snippets with language identifiers (e.g., ```Dockerfile```, ```yaml```, ```ts```). Adjust snippets to match the stack—Next.js examples for frontend, FastAPI examples for backend, Celery for workers, etc.
- **Commands:** Include shell commands and expected outcomes (e.g., ``docker compose up --build`` serves `http://localhost:3000`). Prefix multi-step scripts with `#!/usr/bin/env bash` when asking teams to create new files.
- **Validation Hooks:** Describe how to prove completion (curl endpoints, lint/test commands, coverage thresholds, dashboards). Tie them to acceptance criteria from the epic.
- **Cross-Platform Notes:** If tasks must run on Raspberry Pi or CI, document architecture flags (`DOCKER_DEFAULT_PLATFORM=linux/arm64`) or env overrides.
- **Documentation Updates:** Whenever a task adds or changes docs, reference the exact file (e.g., `docs/architecture/system-architecture.md`) and bullet point what content must be inserted.

If an epic includes UI flows, embed component hierarchy sketches or Storybook story names. For telemetry work, show how to configure exporters and list the metrics/trace names. For CI changes, provide full workflow YAML blocks and highlight secrets/permissions needed.

## 6. Workstream Table Template

Copy/paste this template for every workstream, filling in rows per task. Keep descriptions action-oriented and reference acceptance criteria verbatim when possible.

````markdown
### WSX – <Workstream Name>
| Task ID | Description | Owner | Dependencies | Acceptance |
| --- | --- | --- | --- | --- |
| WSX.1 | <What needs to be delivered and why, referencing requirements doc> | <Role or squad> | <Task IDs, services, or epics> | <Observable outcome (command, test, dashboard)> |
| WSX.2 | ... | ... | ... | ... |
````

Guidelines:
- Use numeric IDs (WS2.1) that match the workstream number to keep traceability.
- Owners can be roles (Frontend, Backend, DevOps) if individuals are unknown.
- Dependencies should include upstream tasks and external artifacts (e.g., “Schema finalized in `docs/requirements/product-requirements.md#designer-workspace`”).
- Acceptance should be testable (“Vitest suite `tests/features/surveys/builder.spec.ts` shows ≥80% coverage”) instead of vague statements.
- After each table, write a `#### WSX Task Details` section mirroring the Epic 0 example: step-by-step instructions, code blocks, commands, and documentation expectations for every task row.

## 7. Verification Checklist

Before submitting the plan, walk through this checklist:
- [ ] Every epic scope bullet and key feature has a corresponding task.
- [ ] Non-functional requirements (performance, security, observability, accessibility) are reflected in at least one task.
- [ ] Workstreams balance frontend, backend, infra, and QA deliverables—no orphaned teams.
- [ ] Milestones are chronological and unblock downstream epics (see dependency lists in `implementation-epics.md`).
- [ ] Risks mention telemetry, data privacy, and tooling gaps noted in `docs/open-questions.md` or `docs/test-notes.md`.
- [ ] Task detail sections include code snippets, commands, and documentation updates ready for handoff.
- [ ] Exit criteria include runnable commands (`npm run dev`, `docker compose up`, tests) so teams can prove completion.

## 8. Hand-Off & Maintenance

1. Commit the new plan file, referencing the epic ID in the commit message (e.g., `docs: add epic-2 plan runbook alignment`).
2. Share the plan with tech leads for review; incorporate their feedback especially around dependencies and acceptance criteria.
3. Update the runbook if the canonical plan format changes. Treat this file as the authoritative process guide and keep it current whenever new requirements or tooling changes land.

Following this runbook ensures every epic plan stays consistent with the SurveyApp requirements, exposes dependencies early, and gives senior engineers a clear starting point for execution.
