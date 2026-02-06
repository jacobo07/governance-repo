# EXECUTION CONTRACT — Universal Provisioning Agent (UPA)

**Version:** 1.0
**Status:** FROZEN
**Constitution Version:** 1.0

---

## Purpose

This document defines the contractual guarantees, explicit non-guarantees, and responsibility boundaries for the UPA execution engine and all parties involved in provisioning operations.

---

## What the UPA Guarantees

### GUARANTEED — Deterministic Behavior

1. **State machine determinism.** Given a valid Canonical Object and a defined trigger, the resulting state transition is always the same.
2. **Tenant isolation at the webhook layer.** Every client tenant receives a webhook path in the format `<tenant-name>-<UUID>`. No two tenants share a webhook path. Template and demo paths use `<UUID>` only.
3. **Schema-driven execution.** The UPA executes only what the provisioning schema defines. No implicit behavior. No feature expansion.
4. **Append-only error logging.** Every failure is logged. Logs are never modified or deleted.
5. **Mandatory manual verification.** No provisioning request reaches COMPLETED without passing through MANUAL_VERIFICATION by a human operator.
6. **Version-governed propagation.** Bug fixes applied to a template are propagated to all active client instances on the same vX.X version. Propagation is logged per tenant.

### GUARANTEED — Structural Integrity

7. **Canonical Object immutability rules.** Immutable fields (request_id, tenant_name, product_type, created_at, schema_ref, operator) are never modified after creation.
8. **Workflow cloning fidelity.** Cloned workflows are structurally identical to their template source at the time of cloning, with only tenant-specific values substituted.
9. **Failure classification.** Every failure is classified as FIXABLE_AUTOMATICALLY or HUMAN_ACTION_REQUIRED before any action is taken.

---

## What the UPA Does NOT Guarantee

### NOT GUARANTEED — External Dependencies

1. **VAPI availability.** The UPA depends on the VAPI external API. If VAPI is down, unreachable, or returns errors, the UPA cannot complete VAPI_CONFIGURING. **FRAGILE.**
2. **CRM/Calendar API availability.** If downstream CRM or calendar services are unavailable, INTEGRATIONS_CONFIGURING will fail.
3. **n8n platform stability.** The UPA runs on n8n. If n8n itself is down or has bugs, the UPA cannot operate.
4. **Credential validity.** The UPA references credentials by ID/name but does not create, rotate, or validate their actual authentication state. Expired credentials will cause failures classified as HUMAN_ACTION_REQUIRED.

### NOT GUARANTEED — Automation Completeness

5. **Fully automated provisioning.** The UPA is NOT fully automated today. Multiple steps require manual operator intervention. See KNOWN_LIMITATIONS.md.
6. **Automated rollback.** Rollback is operator-initiated and operator-verified. The UPA does not auto-rollback on failure.
7. **Automated retry.** Retry is operator-initiated. The UPA does not auto-retry failed provisioning requests.
8. **Partial provisioning recovery.** If provisioning fails mid-pipeline, the UPA does not resume from the point of failure. Retry re-executes from PENDING.

### NOT GUARANTEED — Scope

9. **Multi-product provisioning.** Only Recepcionista IA is supported. No other product types are provisioned.
10. **Client self-service.** Clients cannot initiate, monitor, or manage their own provisioning.
11. **Real-time status updates.** The UPA does not provide real-time provisioning status to anyone except through n8n execution logs.

---

## Responsibility Matrix

### Provisioning Schema

| Responsibility | Owner |
|---|---|
| Define what must be provisioned for each product and version | Schema author (operator/architect) |
| Validate schema completeness before use | UPA (automated) |
| Update schema when product changes | Schema author (operator/architect) |
| Ensure schema does not contradict Constitution | Schema author + UPA lint gate |

The schema is the input contract. If the schema is wrong, the provisioning output will be wrong. The UPA does not infer missing schema elements.

### Execution Engine (UPA Workflows)

| Responsibility | Owner |
|---|---|
| Execute provisioning steps in order | UPA (automated) |
| Track state transitions | UPA (automated) |
| Log all errors | UPA (automated) |
| Classify failures | UPA (automated) |
| Notify operator of outcomes | UPA (automated) |
| Ensure tenant isolation at webhook layer | UPA (automated) |
| Ensure cloning fidelity | UPA (automated) |

The execution engine is deterministic. It does exactly what the schema and state machine dictate. It does not make judgment calls.

### Human Operator

| Responsibility | Owner |
|---|---|
| Initiate provisioning requests | Operator (MANUAL) |
| Verify VAPI configuration | Operator (MANUAL) |
| Verify webhook accessibility | Operator (MANUAL) |
| Confirm MANUAL_VERIFICATION step | Operator (MANUAL) |
| Decide retry vs rollback on FAILED | Operator (MANUAL) |
| Initiate rollback when needed | Operator (MANUAL) |
| Manage credentials (create, rotate, revoke) | Operator (MANUAL) |
| Validate propagation results | Operator (MANUAL) |
| Decide version upgrade timing per client | Operator (MANUAL) |

The human operator is the safety net. The UPA is designed to reduce operator error, not eliminate the operator.

---

## Execution Order Contract

The six specialized workflows execute in this fixed order:

1. Schema Interpretation
2. Workflow Cloning
3. Webhook Assignment
4. Integration Configuration (CRM + Calendar)
5. VAPI Configuration
6. Manual Verification (operator)

This order is not configurable. Skipping a step is not permitted. If any step fails, execution halts and the request transitions to FAILED.

---

## SLA Boundaries

The UPA does not define SLAs. Provisioning time depends on:

- n8n execution speed (variable)
- VAPI API response time (external, uncontrolled)
- Operator availability for manual steps (human-dependent)
- Credential readiness (pre-requisite, not UPA-managed)

No time-based guarantees are made.

---

## Failure Escalation Contract

When a failure occurs:

1. UPA classifies the failure (FIXABLE_AUTOMATICALLY or HUMAN_ACTION_REQUIRED).
2. If FIXABLE_AUTOMATICALLY: UPA performs one deterministic repair and re-validates. If repair fails, reclassify as HUMAN_ACTION_REQUIRED.
3. If HUMAN_ACTION_REQUIRED: UPA logs the failure, notifies the operator with exact remediation steps, and halts. No further automated action.
4. The operator decides the next step: retry, rollback, or manual intervention.

The UPA never silently degrades, never skips a failed step, and never proceeds with partial results.
