# PRD — Universal Provisioning Agent (UPA)

**Version:** 1.0
**Status:** FROZEN
**Product:** NexumOps — Recepcionista IA
**Constitution Version:** 1.0

---

## 1. WHAT

**Product Name:** Universal Provisioning Agent (UPA)
**Domain:** Multi-tenant AI product provisioning for NexumOps
**One sentence:** The UPA is the orchestration engine that provisions, configures, and manages the lifecycle of AI Receptionist instances across NexumOps tenants through a schema-driven, state-machine-governed process.

---

## 2. WHO

**Target user:** NexumOps operations team (internal). The UPA is not customer-facing.
**Their main problem:** Provisioning a new AI Receptionist client requires configuring multiple interdependent systems (VAPI, n8n workflows, webhooks, CRM, calendar) in a specific order with tenant isolation guarantees. Without governance, this process is error-prone, non-repeatable, and unscalable.

---

## 3. WHY

**Why will they pay for this:** The UPA enables NexumOps to sell AI Receptionist instances at 3,500€ setup + 800€/month retainer by ensuring each client deployment is consistent, isolated, and maintainable.
**What are they doing today instead:** A hybrid of manual provisioning steps and partially automated n8n workflows. Many steps require human operator intervention. The UPA formalizes and governs this process.

---

## 4. FEATURES (MAX 5)

1. **Schema-Driven Provisioning** — Interpret a provisioning schema to determine what must be created, configured, and connected for a new tenant.
2. **State Machine Execution** — Track each provisioning request through deterministic states from intake to completion or failure.
3. **Template-to-Client Propagation** — Clone template workflows and configurations into tenant-specific instances following the Template → Demo → Client hierarchy.
4. **Webhook Path Assignment** — Generate and assign tenant-isolated webhook paths using the `<tenant-name>-<UUID>` convention for clients and `<UUID>` for template/demo.
5. **Version-Governed Updates** — Propagate bug fixes from template to all active client instances under vX.X versioning rules.

---

## 5. DATA

### Provisioning Request

| Field | Type | Purpose |
|---|---|---|
| request_id | string (UUID) | Unique identifier for this provisioning request |
| tenant_name | string | Client identifier (e.g., "welcs") |
| product_type | string | Always "recepcionista-ia" for V1 |
| version | string | Target version (e.g., "v2.0") |
| state | enum | Current state in the state machine |
| created_at | ISO 8601 | When the request was created |
| updated_at | ISO 8601 | Last state transition timestamp |
| schema_ref | string | Reference to the provisioning schema used |
| operator | string | Human operator responsible |
| error_log | array | Ordered list of errors encountered |

### Tenant Configuration

| Field | Type | Purpose |
|---|---|---|
| tenant_name | string | Unique tenant identifier |
| webhook_base_path | string | Tenant-specific webhook prefix (`<tenant-name>-<UUID>`) |
| vapi_assistant_id | string | VAPI assistant instance ID |
| workflow_ids | object | Map of provisioned n8n workflow IDs |
| crm_config | object | CRM integration configuration |
| calendar_config | object | Calendar integration configuration |
| version | string | Active product version |
| provisioned_at | ISO 8601 | When provisioning completed |
| status | enum | active, suspended, deprovisioned |

---

## 6. PAGES

The UPA is not a user-facing product. There are no pages or routes.

| Route | Name | Protected |
|---|---|---|
| N/A | N/A | N/A |

The UPA operates entirely through n8n workflows, webhook triggers, and operator-initiated actions.

---

## 7. USER FLOWS

### Flow 1: New Client Provisioning

1. Operator initiates provisioning request with tenant name and target version.
2. UPA validates the provisioning schema for the target product and version.
3. UPA creates provisioning request in PENDING state.
4. UPA executes schema interpretation workflows (six specialized workflows).
5. Each workflow reports success or failure back to the state machine.
6. **MANUAL:** Operator verifies VAPI configuration and credentials.
7. **MANUAL:** Operator confirms webhook paths are accessible.
8. UPA transitions to COMPLETED or FAILED based on workflow outcomes.
9. Operator receives notification of final state.

### Flow 2: Bug Fix Propagation

1. Template workflow is updated with bug fix under current vX.X version.
2. Operator triggers propagation workflow.
3. UPA identifies all active client instances on the affected version.
4. UPA applies the fix to each client instance.
5. **MANUAL:** Operator verifies propagation on at least one client (e.g., Welcs).
6. UPA logs propagation result per tenant.

### Flow 3: Version Upgrade

1. New version (e.g., v3.0) is created in the template tier.
2. Demo instance is provisioned from the new template.
3. **MANUAL:** Operator validates demo behavior end-to-end.
4. Client upgrade is initiated per-tenant by operator decision.
5. UPA provisions new version alongside existing, then operator cuts over.
6. **MANUAL:** Operator confirms client is operational on new version.

---

## 8. INTEGRATIONS

- [x] n8n (workflow execution engine)
- [x] VAPI (voice AI assistant platform)
- [x] Google Sheets (data management and configuration)
- [x] CRM connector (client data sync)
- [x] Calendar integration (appointment management)
- [x] Webhook infrastructure (n8n native)
- [ ] Clerk (not used)
- [ ] Supabase (not used)
- [ ] Stripe (not used — billing is external to UPA)
- [ ] Resend (not used)

---

## 9. DESIGN

Not applicable. The UPA has no user interface. All interaction is through n8n workflow execution, webhook calls, and operator-managed configurations.

- Style: N/A
- Primary color: N/A
- Reference sites: N/A

---

## 10. PRICING

The UPA is internal infrastructure. It does not have its own pricing.

The product it provisions (Recepcionista IA) is priced at:

| Tier | Price | Includes |
|---|---|---|
| Setup | 3,500€ one-time | Full provisioning, configuration, and onboarding |
| Retainer | 800€/month | Ongoing operation, bug fixes, support |

---

## 11. NOT BUILDING (V1)

- Self-service provisioning portal for clients
- Automated credential creation or rotation
- Automated VAPI assistant configuration from scratch
- Multi-product provisioning (only Recepcionista IA)
- Automated billing or payment integration
- Client-facing dashboards or status pages
- Automated rollback (rollback is operator-initiated)
- Auto-scaling or load balancing
- Provisioning for products other than those under Vendibles Individuales → Recepcionista IA
- Internationalization or multi-language provisioning logic

---

## 12. SUCCESS

Success is measured by:

1. **Zero-error first-run provisioning rate** — A new client can be provisioned to operational state with zero n8n import or execution errors.
2. **Tenant isolation guarantee** — No webhook collision, no data leakage between tenants. Binary: pass/fail.
3. **Bug fix propagation completeness** — When a template fix is propagated, 100% of active client instances receive it. Measurable per propagation event.
4. **State machine determinism** — Every provisioning request terminates in a defined terminal state (COMPLETED or FAILED). No requests remain in undefined or stuck states.
5. **Traceability** — Every provisioned artifact is traceable to a provisioning request, a schema version, and an operator.
