# Prompt Compilation Rules

## Scope

This document governs how prompts are constructed for Claude Opus and Claude Code across all agents and systems.

## Rules

1. Every prompt must reference the applicable PRD version and Constitution version.
2. Prompts must specify exactly one execution mode: PRD Review, Architecture, Compilation, or Execution.
3. Prompts must not mix modes. If a task spans multiple modes, it must be split into separate prompts.
4. Prompts must include all context required for execution. No implicit knowledge. No assumed state.
5. Prompts must not contain instructions that conflict with the Constitution.
6. Prompts intended for Claude Code must include the full PRD or an explicit reference to it.
7. Prompts must be token-efficient: no redundant context, no repeated instructions already present in governance documents.
8. Prompts must not instruct Claude to guess, assume, or improvise.
