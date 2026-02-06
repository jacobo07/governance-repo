# Webhook Contract

## Scope

This document governs all webhook-based triggers and endpoints used by any agent or workflow in the system.

## Rules

1. Every webhook must have a defined and documented payload schema.
2. Every webhook must validate incoming payloads before processing.
3. Webhooks must be idempotent: receiving the same payload twice must not produce duplicate side effects.
4. Webhook paths must be deterministic and traceable to a PRD-defined integration.
5. Authentication on webhooks is mandatory for production. No unauthenticated webhooks in production environments.
6. Webhook responses must include appropriate status codes: 200 for success, 400 for invalid payload, 401 for unauthorized, 500 for internal failure.
7. Webhook timeouts must be explicitly configured. No default-timeout reliance.
