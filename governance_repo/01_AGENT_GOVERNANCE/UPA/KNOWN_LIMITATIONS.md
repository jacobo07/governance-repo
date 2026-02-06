# KNOWN LIMITATIONS — Universal Provisioning Agent (UPA)

**Version:** 1.0
**Status:** FROZEN
**Constitution Version:** 1.0

---

## Purpose

This document catalogs the known limitations, fragilities, manual dependencies, and scaling risks of the UPA and the Recepcionista IA product as they exist today. These are facts, not improvement proposals.

---

## Language Limitations

### VAPI and Spanish Language

1. **Primary operating language is Spanish.** The Recepcionista IA serves Spanish-speaking clients in Spain. All voice interactions, booking confirmations, and client-facing communications are in Spanish.
2. **VAPI language model constraints.** VAPI's voice AI capabilities in Spanish are dependent on the underlying LLM and TTS/STT providers. Accent comprehension, regional vocabulary (Andalusian Spanish), and natural-sounding responses may vary in quality. **FRAGILE.**
3. **Prompt engineering in Spanish.** All VAPI assistant prompts must be written and maintained in Spanish. Translation from English-originated patterns may introduce unnatural phrasing. This is a recurring operational concern. **MANUAL — requires native speaker review.**
4. **Error messages and logs in English.** n8n, VAPI, and underlying systems produce error messages in English. The operator must be bilingual (Spanish/English) to debug effectively.

---

## VAPI Constraints

1. **External API dependency.** VAPI is a third-party service. The UPA has no control over VAPI availability, latency, pricing changes, or API breaking changes. **FRAGILE.**
2. **No programmatic assistant creation.** The UPA does not fully automate VAPI assistant creation from scratch. VAPI configuration involves manual steps in the VAPI dashboard or API that are not currently governed by the UPA's provisioning schema. **MANUAL.**
3. **VAPI assistant ID coupling.** Once a VAPI assistant is created and linked to a tenant, the assistant_id becomes immutable in the Canonical Object. Changing it requires deprovisioning and reprovisioning. **FRAGILE.**
4. **Voice quality variability.** VAPI voice output quality depends on the selected voice model and provider. Quality may degrade or change without notice from VAPI's side. **FRAGILE — outside NexumOps control.**
5. **Call duration and concurrency limits.** VAPI may impose limits on simultaneous calls or call duration depending on the plan. These limits are not governed by the UPA. **Uncertainty — depends on VAPI plan terms.**
6. **VAPI webhook delivery.** VAPI sends event webhooks to NexumOps. If VAPI changes its webhook payload structure, NexumOps workflows may break. No schema contract exists between VAPI and NexumOps beyond observed behavior. **FRAGILE.**

---

## Manual Dependencies

The following operations are currently manual and require human operator intervention:

| Operation | Why Manual | Risk if Delayed |
|---|---|---|
| Credential creation and rotation | Security policy: credentials are human-managed only | Provisioning blocked until credentials exist |
| VAPI assistant configuration | No full automation path exists today | Provisioning blocked at VAPI_CONFIGURING |
| Manual verification step (MANUAL_VERIFICATION) | Required by governance: no provisioning completes without human sign-off | Provisioning delayed by operator availability |
| Rollback execution | Rollback involves external systems (VAPI dashboard) not accessible to UPA | Partial artifacts may persist until operator acts |
| Demo validation before propagation | Quality gate: no bug fix reaches clients without human demo review | Bug fix propagation delayed by operator availability |
| Post-propagation client verification | Quality gate: at least one client must be verified after propagation | Undetected regression possible if skipped |
| CRM/Calendar credential configuration per tenant | Per-tenant integration setup requires client-specific credentials | Provisioning blocked at INTEGRATIONS_CONFIGURING |
| Major version upgrade decision per client | Business decision: operator decides when each client upgrades | Clients may stay on older versions indefinitely |
| Deprovisioning and asset cleanup | Involves VAPI dashboard and external service disconnection | Orphaned artifacts in external systems |

---

## Scaling Risks

### Workflow Proliferation

1. **One set of workflows per tenant.** Each client gets approximately six cloned workflows. At 10 clients, the n8n instance hosts ~60 client workflows plus templates and demos. At 50 clients, this grows to ~300+ workflows.
2. **n8n performance under load.** The self-hosted n8n instance may experience performance degradation as workflow count increases. There is no documented scaling threshold. **Uncertainty — not tested at scale.**
3. **Webhook routing at scale.** All tenant webhooks are served by the same n8n instance. High concurrent call volume across many tenants may cause webhook processing delays. **Uncertainty — not tested.**

### Propagation at Scale

4. **Bug fix propagation is linear.** Each tenant is updated individually. At 50+ clients, propagation becomes a time-consuming sequential operation. There is no parallel propagation mechanism. **FRAGILE at scale.**
5. **Manual verification does not scale.** The requirement to verify at least one client after propagation is manageable today (one known client: Welcs). At scale, this becomes a bottleneck or the verification becomes superficial.

### Operational Overhead

6. **Single operator dependency.** Currently, provisioning, verification, rollback, and propagation depend on operator availability. There is no documented backup operator process. **FRAGILE.**
7. **No monitoring dashboard.** There is no centralized view of all tenant statuses, provisioning states, or version distribution. Operator relies on n8n execution history and manual inspection. **MANUAL.**
8. **No alerting system.** If a client's Recepcionista IA stops working (VAPI down, webhook broken, credential expired), there is no automated alert to the operator. Detection depends on client complaint or manual check. **FRAGILE.**

---

## Architectural Limitations

1. **No partial retry.** If provisioning fails at step 4 of 6, retry restarts from step 1. Previously completed steps are re-executed. This is safe (idempotent cloning) but wasteful.
2. **No provisioning queue.** Multiple simultaneous provisioning requests are not explicitly handled. Concurrent provisioning could theoretically cause webhook collision if UUID generation has insufficient entropy. **Low risk but undocumented.**
3. **Single product support.** The UPA currently provisions only Recepcionista IA. Supporting additional products would require schema extensions and potentially new specialized workflows.
4. **No automated testing.** There is no automated test suite for provisioned instances. Verification is entirely manual.
5. **Google Sheets as data layer.** Configuration data managed through Google Sheets introduces latency, concurrency, and reliability risks that a proper database would not have. **FRAGILE at scale.**

---

## Summary Classification

| Limitation | Severity | Mitigation Today |
|---|---|---|
| VAPI external dependency | High | Operator monitoring. No automated failover. |
| Manual verification bottleneck | Medium | Acceptable at current scale (<5 clients). |
| No automated alerting | High | Client complaints are the detection mechanism. |
| Workflow proliferation | Medium | Not yet an issue. Becomes critical at ~50 clients. |
| Linear bug fix propagation | Low | Acceptable at current scale. |
| Single operator dependency | High | No mitigation. Single point of failure. |
| Google Sheets data layer | Medium | Functional today. Risk increases with scale. |
| No partial retry | Low | Full retry is safe, just slower. |
| Spanish language quality in VAPI | Medium | Ongoing prompt tuning. **MANUAL.** |
