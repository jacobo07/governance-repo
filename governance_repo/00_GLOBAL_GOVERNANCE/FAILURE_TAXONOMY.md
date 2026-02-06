# Failure Taxonomy — Prevention First (Auto-Repair Governed)

All known failure classes must be classified before execution.

## Classification Table

| Failure Type | Preventable | Auto-Repair | Human Action |
|---|---|---|---|
| Invalid Switch schema | YES | YES | NO |
| Wrong switch rules key | YES | YES | NO |
| Missing fallback path | YES | YES | NO |
| Array vs Item mismatch | YES | YES | NO |
| Invalid JSON | YES | YES | NO |
| Null / empty data | YES | YES | NO |
| Expression syntax error | YES | YES | NO |
| Infinite loop / unbounded retry | YES | YES | NO |
| Credential expired | NO | NO | YES |
| Invalid API key | NO | NO | YES |
| API down | NO | NO | YES |
| Rate limit exceeded | PARTIAL | PARTIAL | SOMETIMES |
| PRD violation | YES | NO | YES |

## Rules

- If preventable: must be prevented.
- If auto-repairable: must be repaired once, deterministically.
- If human-required: must escalate clearly.
- No silent degradation allowed.

## PRD Failure Class

PRD-related failures are intent definition failures, not execution errors. PRD_VIOLATION includes: missing PRD, incomplete PRD, placeholder content, ambiguous scope, PRD contradicting Constitution, output exceeding PRD scope, feature implemented without PRD authorization. PRD_VIOLATION must be detected before execution, must block the pipeline, must never trigger auto-repair, and requires explicit human clarification.

## Classification Output Contract

Every failure event must emit: failure_class (FIXABLE_AUTOMATICALLY or HUMAN_ACTION_REQUIRED), failure_type, root_cause, repair_plan (if fixable), human_steps (if human-required), changes_applied, validation_after, notification_sent, audit_log_appended.
