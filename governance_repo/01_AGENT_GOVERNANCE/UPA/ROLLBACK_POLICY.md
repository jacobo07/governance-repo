# ROLLBACK POLICY — Universal Provisioning Agent (UPA)

**Version:** 1.0
**Status:** FROZEN
**Constitution Version:** 1.0

---

## Purpose

This document defines when rollback is triggered, what rollback means in the NexumOps operational reality, and the hard boundaries of what can and cannot be reversed.

---

## When Rollback Triggers

Rollback is ALWAYS operator-initiated. The UPA never auto-rolls back. **MANUAL.**

An operator may initiate rollback when:

1. Provisioning fails mid-pipeline and the operator determines that partial artifacts must be cleaned up before retry.
2. Provisioning completes (COMPLETED) but post-deployment verification reveals a critical issue.
3. A version upgrade causes regressions and the operator decides to revert to the previous version.
4. A client is offboarded and all tenant-specific artifacts must be removed.

Rollback is NOT triggered by:

- Transient errors (network timeouts, temporary API failures). These are retried, not rolled back.
- Non-critical configuration issues that can be fixed in place.
- Bug fixes that can be applied without reprovisioning.

---

## What Rollback Means in NexumOps Reality

Rollback is the process of removing or deactivating tenant-specific artifacts created during provisioning. It is a cleanup operation, not a time-travel operation.

Rollback does NOT restore a previous system state. It removes the artifacts associated with a provisioning request so that the system returns to a state as if provisioning had not been attempted.

---

## Rollback Scope — What CAN Be Rolled Back

| Artifact | Rollback Action | Automated |
|---|---|---|
| Cloned n8n workflows | Deactivate and/or delete tenant workflows | MANUAL — operator confirms each |
| Webhook paths | Remove webhook path assignments from tenant configuration | MANUAL — operator verifies no lingering routes |
| CRM configuration | Disconnect CRM integration for tenant | MANUAL — operator handles credential cleanup |
| Calendar configuration | Disconnect calendar integration for tenant | MANUAL — operator handles credential cleanup |
| Provisioning request state | Transition to ROLLED_BACK | YES — state machine transition |
| Tenant configuration object | Mark status as `deprovisioned` | YES — data update |
| Error log entries | NOT rolled back. Preserved for audit. | N/A |

---

## Rollback Scope — What CANNOT Be Rolled Back

| Artifact | Reason |
|---|---|
| VAPI assistant instance | VAPI assistants are managed externally. The UPA does not have delete authority over VAPI resources. Operator must manually deactivate or delete in the VAPI dashboard. **MANUAL.** |
| External CRM records | Data written to a client's CRM during operation cannot be recalled by the UPA. This is outside UPA scope. |
| Calendar events | Appointments booked through the Recepcionista IA while the tenant was active are owned by the client's calendar. Not reversible. |
| Audit log entries | Append-only by design. Rollback does not erase history. |
| Credentials | The UPA does not create or delete credentials. Credential cleanup is always operator-managed. **MANUAL.** |
| Provisioning request object | The request object is never deleted. It transitions to ROLLED_BACK and is preserved. |
| Template workflows | Rollback never affects templates. Templates are shared infrastructure. |
| Demo instances | Rollback of a client does not affect demo instances. |

---

## Rollback Procedure

1. Operator transitions the provisioning request to ROLLBACK_IN_PROGRESS. **MANUAL.**
2. Operator identifies all tenant-specific artifacts using the Canonical Object's tenant_config.
3. Operator deactivates tenant n8n workflows. **MANUAL.**
4. Operator removes or deactivates webhook path assignments. **MANUAL.**
5. Operator disconnects CRM and calendar integrations for the tenant. **MANUAL.**
6. Operator deactivates or deletes the VAPI assistant in the VAPI dashboard. **MANUAL.**
7. Operator updates the tenant_config status to `deprovisioned`.
8. Operator transitions the provisioning request to ROLLED_BACK. **MANUAL.**
9. Rollback is logged in the error_log with: timestamp, reason, artifacts removed, artifacts not removable.

---

## Rollback Failure

If rollback itself fails (e.g., an n8n workflow cannot be deleted, VAPI dashboard is inaccessible):

1. The provisioning request transitions to FAILED (from ROLLBACK_IN_PROGRESS).
2. The error_log is updated with the rollback failure details.
3. The operator must manually complete the cleanup.
4. Once manual cleanup is confirmed, the operator transitions to ROLLED_BACK.

Partial rollback is explicitly allowed — the operator may clean up what is accessible and document what remains. The system does not enforce all-or-nothing rollback.

---

## Invariants

1. Rollback never deletes the provisioning request object. It transitions to a terminal state.
2. Rollback never modifies template or demo instances.
3. Rollback never modifies the error_log except to append rollback-related entries.
4. Rollback never touches other tenants' artifacts.
5. Rollback is always traceable: the Canonical Object records that rollback occurred, when, and by whom.
