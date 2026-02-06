# SECURITY MODEL — Universal Provisioning Agent (UPA)

**Version:** 1.0
**Status:** FROZEN
**Constitution Version:** 1.0

---

## Purpose

This document defines the multi-tenant isolation rules, webhook collision prevention mechanisms, credential handling boundaries, and asset ownership model for the UPA and all provisioned tenant instances.

---

## Multi-Tenant Isolation Rules

### Principle

Every tenant operates in complete isolation from every other tenant. No data, configuration, webhook path, workflow execution, or credential may be shared between tenants unless explicitly governed by global infrastructure (templates).

### Isolation Boundaries

| Layer | Isolation Mechanism | Enforced By |
|---|---|---|
| Webhook paths | Unique `<tenant-name>-<UUID>` per client tenant | UPA during WEBHOOKS_ASSIGNING |
| n8n workflows | Separate cloned workflow instances per tenant | UPA during WORKFLOWS_CLONING |
| VAPI assistants | Separate VAPI assistant instance per tenant | Operator during VAPI configuration. **MANUAL.** |
| CRM connections | Separate credential references per tenant | Operator during integration setup. **MANUAL.** |
| Calendar connections | Separate credential references per tenant | Operator during integration setup. **MANUAL.** |
| Provisioning data | Separate Canonical Object per tenant | UPA (structural) |

### Cross-Tenant Access

Cross-tenant access is forbidden. No workflow, webhook, or integration belonging to tenant A may read, write, or reference data belonging to tenant B.

**Uncertainty:** The current n8n instance hosts all tenant workflows on the same n8n installation. Workflow-level isolation is achieved through separate workflow instances and tenant-specific webhook paths, not through n8n native multi-tenancy (which does not exist in the self-hosted edition used). This is a structural isolation model, not a platform-enforced isolation model.

---

## Webhook Collision Prevention

### Path Convention

| Tier | Path Format | Example |
|---|---|---|
| Template | `<UUID>` | `a1b2c3d4-e5f6-7890-abcd-ef1234567890` |
| Demo | `<UUID>` | `f9e8d7c6-b5a4-3210-fedc-ba0987654321` |
| Client | `<tenant-name>-<UUID>` | `welcs-c3d4e5f6-a1b2-7890-abcd-ef1234567890` |

### Collision Prevention Rules

1. Every webhook path must be globally unique across all tiers (template, demo, client).
2. The UUID component is generated at provisioning time. It is never reused, even if a tenant is deprovisioned and reprovisioned.
3. The `<tenant-name>` prefix in client paths provides human readability but the UUID suffix guarantees uniqueness.
4. Before assigning a webhook path, the UPA must verify that the path does not already exist in any active or suspended tenant configuration.
5. If a collision is detected, the UPA transitions to FAILED. It does not auto-generate an alternative path.

### Webhook Lifecycle

- Webhook paths are assigned during WEBHOOKS_ASSIGNING and become part of the immutable tenant_config.
- Webhook paths are deactivated during rollback or deprovisioning.
- Deactivated webhook paths are never reassigned to new tenants. Path reuse is forbidden.

---

## Credential Handling Boundaries

### Core Rule

The UPA does NOT manage credentials. Credential lifecycle (creation, rotation, revocation) is entirely the responsibility of the human operator.

### What the UPA Does

| Action | Permitted |
|---|---|
| Reference credentials by ID/name in workflow configuration | YES |
| Verify credential existence via n8n API (read-only) | YES |
| Use credentials during workflow execution (via n8n runtime) | YES (indirect — n8n handles this) |

### What the UPA Does NOT Do

| Action | Permitted |
|---|---|
| Create new credentials | NO |
| Modify existing credentials | NO |
| Delete credentials | NO |
| Store credentials in the Canonical Object | NO |
| Rotate or refresh credentials | NO |
| Validate credential authentication state (test if they work) | NO |

### Credential Failure Classification

All credential-related failures are classified as HUMAN_ACTION_REQUIRED. This includes:

- Expired credentials
- Invalid API keys
- Revoked tokens
- Misconfigured OAuth flows
- Missing credentials referenced by a workflow

The UPA will not auto-repair credential failures under any circumstances.

---

## Asset Ownership Model

### NexumOps-Owned Assets

| Asset | Ownership | Tenant Access |
|---|---|---|
| Template workflows | NexumOps (global) | None. Templates are internal infrastructure. |
| Demo instances | NexumOps (global) | None. Demos are internal. |
| Provisioning schemas | NexumOps (global) | None. Schemas are internal. |
| UPA engine workflows | NexumOps (global) | None. Engine is internal. |
| n8n instance | NexumOps (infrastructure) | None. Clients do not access n8n. |
| Governance repository | NexumOps (global) | None. |

### Tenant-Owned Assets (Post-Provisioning)

| Asset | Ownership | Notes |
|---|---|---|
| Client-specific n8n workflows | NexumOps manages, client benefits | Client does not directly access workflows. NexumOps operates them on behalf of the client. |
| Client webhook paths | NexumOps manages | Client's external systems may call these paths. |
| Client VAPI assistant | NexumOps manages | Configured per client's business requirements. |
| Client CRM data | Client owns | Data in the client's CRM system belongs to the client. NexumOps has integration access only. |
| Client calendar data | Client owns | Appointments belong to the client. NexumOps has integration access only. |
| Call recordings (if any) | Subject to client agreement | Ownership depends on service agreement. **Uncertainty — not governed by UPA.** |

### Deprovisioning and Asset Disposition

When a tenant is deprovisioned:

1. NexumOps-managed assets (workflows, webhooks, VAPI config) are deactivated and/or deleted.
2. Client-owned assets (CRM data, calendar events) remain with the client. NexumOps integration access is revoked.
3. The Canonical Object is preserved in DEPROVISIONED state for audit purposes.

---

## Security Invariants

1. No tenant's webhook path may collide with any other tenant's, template's, or demo's webhook path.
2. No tenant's workflow may reference another tenant's data, configuration, or credentials.
3. Credentials are never stored in provisioning objects, logs, or governance documents.
4. Credential lifecycle is always human-managed.
5. Deprovisioned tenant paths are never reused.
6. All security-relevant events (provisioning, rollback, deprovisioning) are logged in the append-only audit trail.
