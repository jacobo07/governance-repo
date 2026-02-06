# Governance Principles

These principles govern all decisions, designs, and outputs across the system.

## Hierarchy

1. Governance over Product — governance constraints override product desires.
2. Reuse over Duplication — shared contracts are preferred to agent-specific rewrites.
3. Contracts over Code — behavior is defined by governance documents, not by implementation details.
4. Explicit over Implicit — every boundary, limitation, and requirement must be stated.
5. Deterministic over Flexible — systems must behave predictably under all conditions.

## Scope Control

- Features are bounded by PRD. Anything not in the PRD does not exist.
- NOT BUILDING sections are hard limits, not suggestions.
- No scope widening is permitted without a new PRD version and full pipeline restart.

## Traceability

Every workflow, node, integration, and data entity must be traceable to a PRD feature, a PRD user flow, or a PRD integration. Untraceable elements are invalid.

## Immutability

Once execution begins, the PRD is frozen. No silent edits to scope, features, data, or flows are allowed. Any change requires a new PRD version and full pipeline restart from Step 0.

## Agent Independence

Each agent operates under its own PRD and governance documents within the 01_AGENT_GOVERNANCE directory. Global governance applies to all agents without exception. Agent-specific governance may add constraints but may never relax global constraints.

## Versioning

All governance documents, PRDs, and execution artifacts must be versioned. Outputs must be traceable to the PRD version and Constitution version that governed their creation.
