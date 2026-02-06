# CANONICAL OBJECT — Universal Provisioning Agent (UPA)

**Version:** 1.0
**Status:** FROZEN
**Constitution Version:** 1.0

---

## Purpose

The Canonical Provisioning Object is the single authoritative data structure that represents a provisioning request and its associated tenant configuration throughout the UPA lifecycle. Every workflow, state transition, and audit event references this object. No provisioning action may occur without a valid Canonical Object.

---

## Provisioning Request Object

```json
{
  "request_id": "string (UUID)",
  "tenant_name": "string",
  "product_type": "string",
  "version": "string",
  "state": "string (enum)",
  "created_at": "string (ISO 8601)",
  "updated_at": "string (ISO 8601)",
  "schema_ref": "string",
  "operator": "string",
  "error_log": [],
  "tenant_config": {}
}
```

---

## Field Definitions and Invariants

### Immutable Fields (set once, never modified)

| Field | Type | Set When | Invariant |
|---|---|---|---|
| request_id | UUID string | Object creation | Globally unique. Never reused. Never null after creation. |
| tenant_name | string | Object creation | Must match `^[a-z0-9\-]+$`. Cannot be changed after creation. Must be unique across all active tenants. |
| product_type | string | Object creation | Always `"recepcionista-ia"` in V1. No other values permitted. |
| created_at | ISO 8601 string | Object creation | Set once. Never modified. |
| schema_ref | string | Object creation | Reference to the provisioning schema version used. Immutable once set. |
| operator | string | Object creation | Identifier of the human operator who initiated the request. |

### Mutable Fields (change during lifecycle)

| Field | Type | Mutation Rules |
|---|---|---|
| version | string | Set at creation. May change during version upgrade flow only. Must follow `vX.X` format. |
| state | enum | Changes only via valid state machine transitions. See STATE_MACHINE.md. |
| updated_at | ISO 8601 string | Updated on every state transition. Automatically set. |
| error_log | array of objects | Append-only. Entries are never modified or deleted. |
| tenant_config | object | Populated progressively during provisioning. See below. |

---

## Tenant Configuration Sub-Object

```json
{
  "webhook_base_path": "string",
  "vapi_assistant_id": "string",
  "workflow_ids": {
    "main": "string",
    "booking": "string",
    "rescheduling": "string",
    "cancellation": "string",
    "crm_sync": "string",
    "call_logging": "string"
  },
  "crm_config": {
    "type": "string",
    "connection_id": "string"
  },
  "calendar_config": {
    "type": "string",
    "connection_id": "string"
  },
  "version": "string",
  "provisioned_at": "string (ISO 8601)",
  "status": "string (enum)"
}
```

### Tenant Configuration Field Rules

| Field | Mutability | Invariant |
|---|---|---|
| webhook_base_path | Immutable after assignment | Format: `<tenant-name>-<UUID>` for clients. `<UUID>` for template/demo. Must be globally unique. |
| vapi_assistant_id | Immutable after assignment | References a real VAPI assistant instance. Validated externally. **FRAGILE — depends on VAPI API.** |
| workflow_ids | Immutable after provisioning | Each key maps to a real n8n workflow ID. All six must be populated for COMPLETED state. |
| crm_config | Mutable by operator only | Connection ID references a real n8n credential. **MANUAL to change.** |
| calendar_config | Mutable by operator only | Connection ID references a real n8n credential. **MANUAL to change.** |
| version | Mutable during upgrade only | Must match the request-level version after upgrade completes. |
| provisioned_at | Immutable after set | Set when state transitions to COMPLETED. |
| status | Mutable | Valid values: `active`, `suspended`, `deprovisioned`. Transitions governed by state machine. |

---

## Error Log Entry Structure

```json
{
  "timestamp": "string (ISO 8601)",
  "source_state": "string",
  "failed_node": "string",
  "error_message": "string",
  "payload_shape": "string",
  "classification": "string (FIXABLE_AUTOMATICALLY | HUMAN_ACTION_REQUIRED)"
}
```

### Error Log Invariants

1. The error_log is append-only. No entry may be modified or removed.
2. Every state transition to FAILED must append at least one error log entry.
3. Error log entries must include all six fields. No nulls permitted.
4. The error_log persists across retries. Retry does not clear previous errors.

---

## Validation Rules (Pre-Execution)

Before the UPA begins processing a provisioning request, the Canonical Object must satisfy:

1. `request_id` is a valid UUID and not null.
2. `tenant_name` matches `^[a-z0-9\-]+$` and is unique among active tenants.
3. `product_type` equals `"recepcionista-ia"`.
4. `version` matches `^v\d+\.\d+$`.
5. `state` is `PENDING`.
6. `created_at` is a valid ISO 8601 timestamp.
7. `schema_ref` is not null and references an existing provisioning schema.
8. `operator` is not null.
9. `error_log` is an empty array.
10. `tenant_config` is an empty object (populated during execution).

If any validation fails, the request must not proceed. Classification: HUMAN_ACTION_REQUIRED.

---

## Uniqueness Guarantees

| Scope | Field | Guarantee |
|---|---|---|
| Global | request_id | No two provisioning requests share a request_id. |
| Active tenants | tenant_name | No two active (non-deprovisioned) tenants share a name. |
| Global | webhook_base_path | No two tenants (including template/demo) share a webhook path. |
| Per tenant | workflow_ids values | Each workflow ID within a tenant is distinct. |

---

## Object Lifecycle

1. **Created** — When operator initiates provisioning. State: PENDING. tenant_config: empty.
2. **Populated** — During active states. tenant_config fields are set progressively as workflows execute.
3. **Frozen** — When state reaches COMPLETED. The object is considered immutable except for status changes (suspend/deprovision) and version upgrades.
4. **Archived** — When state reaches DEPROVISIONED. The object is retained for audit but no further mutations are permitted.
