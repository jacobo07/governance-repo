# Credential Handling

## Scope

This document governs how credentials are referenced, validated, and protected across all execution environments.

## Rules

1. Credentials are NEVER created, modified, or deleted by automated systems. These are human-only operations.
2. Credentials are referenced by ID or name only. No inline secrets, API keys, or tokens in any workflow, prompt, or governance document.
3. Before any workflow build, credential existence must be confirmed via read-only API or MCP query.
4. Expired or invalid credentials are classified as HUMAN_ACTION_REQUIRED. No auto-repair is permitted.
5. Credential IDs must be traceable to a PRD-defined integration.
6. No credential may be shared across agents unless explicitly authorized in global governance.
