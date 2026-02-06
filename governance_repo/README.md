# Governance Repository

This repository is the single source of truth for governance, contracts, PRDs, constraints, and execution laws used by Claude Opus and Claude Code across all agents, products, and execution environments.

Every agent, workflow, pipeline, and system artifact must be traceable to a governance document in this repository. No system may be designed, compiled, or executed without a valid PRD that has been reviewed against the Constitution and all applicable governance rules.

This structure is designed for reuse without refactor. New agents are created by copying _TEMPLATE_AGENT and populating the required documents. Global governance applies universally and may not be overridden by any agent-specific document.
