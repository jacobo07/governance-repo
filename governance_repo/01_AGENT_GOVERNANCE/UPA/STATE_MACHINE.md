# STATE MACHINE — Universal Provisioning Agent (UPA)

**Version:** 1.0
**Status:** FROZEN
**Constitution Version:** 1.0

---

## States

| State | Type | Description |
|---|---|---|
| PENDING | Initial | Provisioning request received and validated. No execution has started. |
| SCHEMA_INTERPRETING | Active | The provisioning schema is being read and interpreted by the six specialized workflows. |
| WORKFLOWS_CLONING | Active | Template workflows are being cloned into tenant-specific instances. |
| WEBHOOKS_ASSIGNING | Active | Tenant-isolated webhook paths are being generated and assigned. |
| INTEGRATIONS_CONFIGURING | Active | CRM, calendar, and external service integrations are being configured for the tenant. |
| VAPI_CONFIGURING | Active | VAPI assistant is being linked and configured for the tenant. **FRAGILE — depends on VAPI external API availability.** |
| MANUAL_VERIFICATION | Waiting | Operator must manually verify the provisioned instance. System is blocked until operator confirms. **MANUAL.** |
| COMPLETED | Terminal | Provisioning succeeded. All artifacts created, verified, and logged. |
| FAILED | Terminal | Provisioning failed. Error log populated. Operator must review and decide on retry or abort. |
| ROLLBACK_IN_PROGRESS | Active | A previously completed or partially completed provisioning is being reversed. **MANUAL — operator-initiated only.** |
| ROLLED_BACK | Terminal | Rollback completed. Tenant artifacts removed or deactivated. |
| SUSPENDED | Terminal | Tenant is provisioned but service is suspended (e.g., non-payment, client request). |
| DEPROVISIONED | Terminal | Tenant has been fully deprovisioned. All artifacts removed. |

---

## Valid Transitions

| From | To | Trigger | Automated |
|---|---|---|---|
| PENDING | SCHEMA_INTERPRETING | Operator initiates provisioning | YES |
| SCHEMA_INTERPRETING | WORKFLOWS_CLONING | Schema interpretation completes successfully | YES |
| SCHEMA_INTERPRETING | FAILED | Schema interpretation fails (invalid schema, missing fields) | YES |
| WORKFLOWS_CLONING | WEBHOOKS_ASSIGNING | All template workflows cloned successfully | YES |
| WORKFLOWS_CLONING | FAILED | Cloning fails (n8n API error, template not found) | YES |
| WEBHOOKS_ASSIGNING | INTEGRATIONS_CONFIGURING | Webhook paths assigned and confirmed unique | YES |
| WEBHOOKS_ASSIGNING | FAILED | Webhook collision detected or assignment fails | YES |
| INTEGRATIONS_CONFIGURING | VAPI_CONFIGURING | CRM and calendar configured successfully | YES |
| INTEGRATIONS_CONFIGURING | FAILED | Integration configuration fails | YES |
| VAPI_CONFIGURING | MANUAL_VERIFICATION | VAPI assistant linked successfully | YES |
| VAPI_CONFIGURING | FAILED | VAPI API error, timeout, or configuration failure | YES |
| MANUAL_VERIFICATION | COMPLETED | Operator confirms all systems operational | MANUAL |
| MANUAL_VERIFICATION | FAILED | Operator identifies issues during verification | MANUAL |
| FAILED | PENDING | Operator decides to retry after fixing root cause | MANUAL |
| FAILED | ROLLBACK_IN_PROGRESS | Operator decides to rollback | MANUAL |
| COMPLETED | SUSPENDED | Operator suspends tenant service | MANUAL |
| COMPLETED | ROLLBACK_IN_PROGRESS | Operator initiates rollback of completed provisioning | MANUAL |
| SUSPENDED | COMPLETED | Operator reactivates tenant service | MANUAL |
| SUSPENDED | DEPROVISIONED | Operator decommissions tenant | MANUAL |
| ROLLBACK_IN_PROGRESS | ROLLED_BACK | All reversible artifacts removed or deactivated | MANUAL |
| ROLLBACK_IN_PROGRESS | FAILED | Rollback itself fails (partial cleanup) | MANUAL |

---

## Invalid Transitions (Explicitly Forbidden)

- COMPLETED → PENDING (cannot re-provision without new request)
- ROLLED_BACK → any active state (rolled-back requests are terminal)
- DEPROVISIONED → any state (deprovisioned tenants are terminal)
- Any active state → COMPLETED (must pass through MANUAL_VERIFICATION)
- Any state → SCHEMA_INTERPRETING without passing through PENDING

---

## Terminal States

| State | Meaning | Recovery Path |
|---|---|---|
| COMPLETED | Success. Tenant is operational. | N/A |
| FAILED | Failure. Requires human review. | Retry (→ PENDING) or Rollback (→ ROLLBACK_IN_PROGRESS) |
| ROLLED_BACK | Reversed. Artifacts cleaned up. | New provisioning request required. |
| SUSPENDED | Service paused. Artifacts exist but inactive. | Reactivate (→ COMPLETED) or Deprovision (→ DEPROVISIONED) |
| DEPROVISIONED | Permanently removed. | No recovery. New provisioning request required. |

---

## Retry Rules

1. Retry is permitted ONLY from the FAILED state.
2. Retry transitions the request back to PENDING, not to the state where failure occurred.
3. Retry resets the execution from the beginning of the pipeline (full re-run).
4. There is no partial retry. The UPA does not resume from mid-pipeline.
5. Maximum retry count is not enforced by the system. Operator judgment applies. **MANUAL.**
6. Each retry attempt is logged in the error_log with the previous failure context preserved.

---

## Failure Handling

- Every state transition that results in FAILED must populate the error_log with: timestamp, source state, failed node/workflow, error message, payload shape summary.
- The state machine does not auto-recover. All FAILED states require human review. **MANUAL.**
- The only automated failure response is classification (FIXABLE_AUTOMATICALLY vs HUMAN_ACTION_REQUIRED) and notification to the operator.

---

## Invariants

1. A provisioning request is in exactly one state at any time.
2. No state transition occurs without logging.
3. No request may reach COMPLETED without passing through MANUAL_VERIFICATION.
4. No request may bypass PENDING as its initial state.
5. Terminal states are final unless an explicit recovery path is defined above.
6. The state machine is deterministic: given a state and a trigger, the next state is always the same.
