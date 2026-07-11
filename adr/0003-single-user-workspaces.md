# ADR-0003: Single-user workspaces for MVP

- **Status:** Accepted
- **Date:** 2026-07-11

## Context

The product could support collaboration (shared folders, shared documents) or remain
strictly single-user for the MVP. Collaboration introduces authorization complexity
(ownership vs. membership, permission levels, sharing invitations) that would slow down
learning the core backend fundamentals.

## Decision

For the MVP, every workspace is **single-user**. A user can only ever see and act on their
own folders and documents. Every data-access path is scoped to the authenticated user's id.

## Consequences

**Positive**
- Authorization stays simple: "does this resource belong to the current user?"
- Faster path to a working, deployable product.
- Still demonstrates real access-control discipline (owner-scoped queries, no data leakage
  across users).

**Negative / tradeoffs**
- No sharing/collaboration until a later phase.
- The data model should not *prevent* future sharing; we avoid choices that would make
  adding membership/permissions painful later.

## Notes

Sharing and collaboration are tracked for Phase 16 (Future Enhancements).
