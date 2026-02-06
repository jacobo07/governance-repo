# n8n Execution Rules

## Schema Truth

The n8n runtime is the single source of truth for all workflow artifacts. Node schemas must be fetched from n8n API or MCP before any build. No node parameters may be invented. No switch, code, or IF node schema variants may be assumed.

## Build Rules

- No pseudo-code, placeholders, TODOs, or partial JSON.
- All workflows must include fallback paths, input sanitization at boundaries, idempotency where applicable, and deterministic routing.
- Every workflow must pass: workflow validation, connection validation, expression validation, and lint checks before delivery.

## Error Workflow Requirement

Every production workflow must have an Error Workflow configured that captures workflow_id, execution_id, failed node, error_message, and payload shape summary. The error workflow must classify failures as FIXABLE_AUTOMATICALLY or HUMAN_ACTION_REQUIRED and act accordingly.

## Import Safety

Workflow JSON must be complete. typeVersion must be compatible with the target n8n instance. No invalid fields. Connection array indexes must be valid.

## Credential Handling

Workflows may reference only credentials that exist by ID or name. No placeholder credentials are allowed in production. Credential validation is read-only; Claude Code must never create, modify, or delete credentials.
