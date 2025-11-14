# User Stories Backlog

## Survey Designer
- As a designer, I can create a survey with metadata (title, description, category, tags, closing date, owner) so stakeholders understand its purpose and where it fits in the program.
  - **Acceptance:** metadata form enforces required fields, validates closing date > today, shows owner dropdown, and persists draft immediately.
- As a designer, I can add questions of varying types (choice, scale, numeric, etc.) so I capture the right data.
  - **Acceptance:** each question type exposes validation settings (required, min/max, regex), tooltips explain use cases, and builder enforces <200 characters for prompts by default.
- As a designer, I can preview the respondent experience before publishing so I validate branching and formatting.
  - **Acceptance:** preview respects branching logic, section randomization, translations, and file upload limits, and shows PII warnings if any questions collect sensitive data.
- As a designer, I can clone an existing survey so I reuse structure without rebuilding every question.
  - **Acceptance:** clone copies sections, branching rules, metadata, and question bank references, but resets closing date and distribution lists.
- As a designer, I can import recipients from CSV and schedule delivery windows so I can plan launches across time zones.
  - **Acceptance:** uploader validates mandatory `email` column, displays errors per row, and wizard requests start/end datetime in workspace timezone.
- As a designer, I can define question bank templates and reuse them across surveys so I maintain consistency.
  - **Acceptance:** editing a bank question prompts whether new surveys should inherit future edits or remain frozen.

## Administrator
- As an admin, I can view an overview dashboard showing total responses, completion rate, runtime anomalies, and closing date countdown for each survey so I can intervene early.
  - **Acceptance:** dashboard refreshes within 30 seconds, visually flags surveys with <50% completion and <3 days remaining, and links to detail pages.
- As an admin, I can drill into an individual survey’s analytics so I can view response distributions, dropout sections, and export results to CSV.
  - **Acceptance:** filters by recipient segment, honors RBAC permissions, CSV export triggers background job with status toast + email link.
- As an admin, I can pause or close a survey early so I can prevent further responses when issues arise.
  - **Acceptance:** pausing immediately invalidates tokens, logs audit entry, and notifies designers.
- As an admin, I can manage workspace members and roles so access stays least-privilege.
  - **Acceptance:** roles include Designer, Analyst, Admin with granular scopes; changes sync to audit log and propagate to back-end token on next refresh.
- As an admin, I can monitor scheduled exports and warehouse sync jobs so I ensure downstream analytics are current.
  - **Acceptance:** job list shows status, duration, retries, and allows manual rerun with guardrail for concurrent executions.

## Respondent
- As a respondent, I can open my unique link without creating an account so I can quickly answer.
  - **Acceptance:** expired/invalid links show friendly error with support contact, valid links preload recipient name and optional locale.
- As a respondent, I can save progress and resume before the closing date so longer surveys don’t require a single sitting.
  - **Acceptance:** progress auto-saves every 10 seconds or on section change; resuming on new device reenforces token via email PIN.
- As a respondent, I can see confirmation after submission so I know my responses were stored.
  - **Acceptance:** confirmation page provides timestamp, optional downloadable receipt, and suppression of edit controls if editing disabled.
- As a respondent, I can upload supporting files (e.g., receipts) when questions allow attachments so I can provide proof.
  - **Acceptance:** upload enforces size/type limits, scans for viruses (placeholder), and shows upload progress + retry.
- As a respondent, I can switch languages when translation packs exist so I can answer in my preferred language.
  - **Acceptance:** language toggle persists in local storage and re-renders translated labels immediately.

## Technical/Platform
- As a developer, I can reference environment variable contracts so I configure frontend, backend, and workers consistently.
  - **Acceptance:** docs enumerate required vars, defaults, and rotation cadence; CI fails if required vars missing.
- As an operator, I can view audit logs of survey and admin actions so compliance reviews are possible.
  - **Acceptance:** audit log exposes filters (actor, date, entity), export option, and includes hash/signature of payload for tamper detection.
- As a support engineer, I can impersonate a designer within support tooling (without seeing respondent data) so I can troubleshoot builder issues safely.
  - **Acceptance:** impersonation requires elevated role, logs reason, and times out after 15 minutes.

> _Add INVEST acceptance criteria under each story as details solidify. Consider labeling each story with priority (P0/P1/P2) and linking to Jira or GitHub issues once tracking begins._
