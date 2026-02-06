# VERSIONING RULES — Universal Provisioning Agent (UPA)

**Version:** 1.0
**Status:** FROZEN
**Constitution Version:** 1.0

---

## Purpose

This document defines the versioning model for the Recepcionista IA product as managed by the UPA, including the Template → Demo → Client propagation hierarchy and the rules governing bug fix distribution.

---

## Version Format

All versions follow the `vX.X` format (e.g., v1.0, v2.0, v2.1).

| Component | Meaning |
|---|---|
| X (major) | Significant feature change or architectural revision. Requires new provisioning. |
| X (minor) | Bug fix, configuration adjustment, or incremental improvement. Propagated to existing clients. |

### Rules

1. Version strings must match `^v\d+\.\d+$`. No other formats are permitted.
2. Major version increments (e.g., v1.x → v2.0) represent breaking changes. Existing clients are not auto-upgraded.
3. Minor version increments (e.g., v2.0 → v2.1) represent non-breaking changes. Existing clients on the same major version receive the update.
4. Version is recorded in the Canonical Object at provisioning time and updated only during explicit version upgrade flows.

---

## Propagation Hierarchy

The version hierarchy flows in one direction: Template → Demo → Client.

```
Template (source of truth)
    ↓
Demo (validation instance)
    ↓
Client(s) (production instances)
```

### Template Tier

- The template is the canonical reference for each product version.
- All changes originate at the template level.
- The template is never client-specific. It contains no tenant data, no tenant webhook paths, and no tenant credentials.
- Template workflows live under: NexumOps → Sistemas IA → Vendibles → Vendibles Individuales → Recepcionista IA → Template.

### Demo Tier

- The demo is a functional instance cloned from the template for internal validation.
- Demo webhook paths use the `<UUID>` format (no tenant prefix).
- Demo instances are used to verify changes before propagating to clients.
- Demo must be validated by the operator before any client propagation. **MANUAL.**
- Demo workflows live under: NexumOps → Sistemas IA → Vendibles → Vendibles Individuales → Recepcionista IA → Demo.

### Client Tier

- Client instances are cloned from the template (not from demo).
- Client webhook paths use `<tenant-name>-<UUID>` format.
- Client instances receive bug fixes through the propagation mechanism described below.
- Client instances are NOT auto-upgraded across major versions.
- Client workflows live under: NexumOps → Sistemas IA → Vendibles → Vendibles Individuales → Recepcionista IA → Clientes → `<tenant-name>`.

---

## Bug Fix Propagation (Global)

### Definition

A bug fix is any change to the template that increments the minor version (e.g., v2.0 → v2.1) and corrects defective behavior without changing the product's feature set or interface.

### Propagation Rules

1. The bug fix is applied first to the template.
2. The demo instance is updated from the template.
3. The operator validates the demo. **MANUAL.**
4. Once validated, the fix is propagated to ALL active client instances on the affected major version.
5. Propagation is per-tenant: each client instance is updated individually.
6. Each propagation event is logged with: tenant_name, previous_version, new_version, timestamp, operator.
7. After propagation, the operator verifies at least one client instance (e.g., Welcs). **MANUAL.**
8. The Canonical Object's version field is updated for each affected tenant.

### Propagation Scope

| What is propagated | What is NOT propagated |
|---|---|
| Workflow logic changes (node configurations, expressions, routing) | Tenant-specific webhook paths |
| Corrected code node logic | Tenant-specific credential references |
| Fixed switch/if conditions | Tenant-specific CRM/calendar configuration |
| Updated error handling | Tenant-specific VAPI assistant IDs |

Bug fix propagation modifies workflow behavior. It never modifies tenant identity, paths, or integration bindings.

### Propagation Failure

If propagation fails for a specific tenant:

1. The failure is logged in that tenant's error_log.
2. Other tenants are not affected — propagation continues independently per tenant.
3. The failed tenant remains on the previous version until the operator resolves the issue. **MANUAL.**
4. The operator is notified with the exact failure details and remediation steps.

---

## Major Version Upgrade

### Definition

A major version upgrade (e.g., v1.x → v2.0) introduces significant changes that may alter features, interfaces, data models, or integration requirements.

### Upgrade Rules

1. Major version upgrades are NEVER automatic. They are always operator-initiated per client. **MANUAL.**
2. The new version must exist as a fully validated template before any client upgrade begins.
3. The demo must be provisioned on the new version and validated before any client upgrade. **MANUAL.**
4. Client upgrade is performed by provisioning a new instance on the new version alongside the existing one, then cutting over. This is not an in-place upgrade.
5. The old instance is deprovisioned after the operator confirms the new instance is operational. **MANUAL.**
6. The Canonical Object for the old instance transitions to DEPROVISIONED. A new Canonical Object is created for the new version instance.

---

## Version Coexistence

Multiple major versions may coexist in production. For example:

- Welcs: v2.0
- Future client A: v2.1
- Future client B: v3.0

Bug fixes propagate only within the same major version line. A v2.1 fix does not affect v3.0 clients and vice versa.

---

## Invariants

1. All active clients are on a version that exists as a template.
2. No client runs a version that has not been validated through demo first.
3. Bug fix propagation never skips the demo validation step.
4. Version strings are never reused. Once v2.1 exists, it is permanent. If it needs correction, it becomes v2.2.
5. The template is the single source for all client cloning. Clients are never cloned from other clients.
6. The propagation hierarchy is strictly one-directional: Template → Demo → Client. Reverse flow is forbidden.
