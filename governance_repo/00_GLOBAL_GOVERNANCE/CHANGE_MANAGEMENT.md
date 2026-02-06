# Change Management

## Scope

This document governs all changes to governance documents, PRDs, agent configurations, execution environments, and Claude interface contracts.

## Rules

1. No governance document may be modified without explicit authorization from the repository governor.
2. All changes must be versioned with a clear diff of what changed and why.
3. Changes to the Constitution require full review and revalidation of all active PRDs.
4. Changes to agent-specific governance require revalidation of that agent's PRD only.
5. Changes to execution environment rules require revalidation of all workflows operating in that environment.

## PRD Change Protocol

- Once a PRD enters execution, it is frozen.
- Any change to a frozen PRD requires a new PRD version.
- A new PRD version triggers a full pipeline restart from Step 0.
- The old PRD version must be preserved for audit traceability.

## Rollback

- Any change that causes a downstream failure must be rolled back.
- Rollback means reverting to the last known-good version of the changed document.
- Rollback does not require a new PRD version; it restores the previous one.

## Audit

- Every change must be logged with: document changed, previous version, new version, reason, authorized by, timestamp.
- The audit log is append-only and may not be modified or deleted.
