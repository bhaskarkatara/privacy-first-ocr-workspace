# ADR-0004: Deletion is a hard delete, not soft delete/trash

- **Status:** Accepted
- **Date:** 2026-07-12

## Context

When a user deletes a document, the system could either permanently remove it (hard delete)
or mark it as deleted while retaining it (soft delete / trash), enabling an "undo" or
recovery window.

## Decision

Deletion is a **hard delete**. When a user deletes a document, the record is permanently
removed from storage.

## Consequences

**Positive**
- Consistent with the product's privacy stance: the system does not quietly retain data a
  user has asked to remove. "Delete" means what it says.
- Simpler data model and query logic — no `deleted_at`/`is_deleted` filtering needed
  everywhere.

**Negative / tradeoffs**
- No undo. A mis-click is unrecoverable. Mitigated at the UX layer with a confirmation step
  before delete (frontend concern, not backend).
- Forecloses a future "trash" feature without a later migration.

## Notes

Revisit if user research later shows accidental deletion is a frequent pain point; a soft
delete with a short retention window could be added as an explicit, visible feature rather
than silent retention.
