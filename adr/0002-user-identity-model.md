# ADR-0002: User identity as one User with multiple credential records

- **Status:** Accepted (implementation details finalized in Phase 5 and Phase 7)
- **Date:** 2026-07-11

## Context

MVP supports both email/password (with JWT) and Google OAuth2 login. This raises the
account-linking problem: a person may register with a password using an email, then later
sign in with Google using the same email. We must decide whether a `User` represents one
human or one login method.

Naive approaches are dangerous:
- Creating a second account for the Google login splits one person's data across two
  accounts.
- Auto-linking a Google login to an existing account by email alone enables account
  takeover: an attacker pre-registers a password account with a victim's email, and when
  the victim later signs in with real Google, they land in the attacker's account.

## Decision

Model a `User` as **one human**, with **one or more attached credential/identity records**:
- a `LOCAL` credential holding the password hash, and/or
- a `GOOGLE` identity holding the provider's stable subject id (`sub`).

Auto-link a new login method to an existing `User` **only when the email is verified on both
sides** — the OAuth provider asserts `email_verified: true` *and* the local account's email
was confirmed. Otherwise, require an explicit, authenticated linking step.

## Consequences

**Positive**
- One person = one account, with clean support for multiple sign-in methods.
- Closes the email-based account-takeover vector.
- Extends naturally to additional OAuth providers later.

**Negative / tradeoffs**
- More schema and logic than a single `password_hash` + `google_id` on the `User` row.
- Requires an email-verification mechanism to satisfy the linking rule (scope confirmed in
  Phase 5/7).

## Notes

Concrete table design lands in Phase 5 (Database Design); the security flows (verification,
linking, JWT vs OAuth2 handling) land in Phase 7 (Security Design).
