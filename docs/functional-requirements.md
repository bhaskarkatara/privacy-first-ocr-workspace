# Functional Requirements (Phase 2)

> Format: user stories in "As a [role], I want [capability], so that [benefit]" form, each
> with acceptance criteria. Priority uses MoSCoW (Must / Should / Could / Won't).
> OCR itself is a frontend-only capability (no backend involvement) and is listed for
> completeness only; everything else here is backend-facing.

## Two distinct user behaviors (why saving exists)

Extraction has two different shapes of use, and the product serves both:

- **One-off use** — a visitor OCRs a single image and just wants the text right now, to
  paste elsewhere. No account, nothing saved. Served by a client-side "copy to clipboard"
  action with zero backend involvement.
- **Accumulation use** — a user extracts text repeatedly over time (receipts, notes,
  documents) and wants a searchable, organized, cross-device library of that text. A single
  saved document has little value; the library built from many does. This is what the
  authenticated save/folder/search features exist to serve.

Both are first-class; neither is forced on the other's user.

## Epic A — Authentication & Account

| # | Story | Priority |
|---|---|---|
| A1 | Register with email + password | Must |
| A2 | Log in with email + password, receive a session token | Must |
| A3 | Log in with Google | Must* |
| A4 | Password + Google on the same verified email resolve to one account | Must* |
| A5 | Log out | Must |
| A6 | Verify email | Should |

*\*Built as the second increment, after password+JWT lands, per the walking-skeleton plan.*

### A1 — Register with email/password
*As a visitor, I want to register with email and password, so that I can save my work.*
**Acceptance criteria:**
- Given a valid, not-already-registered email and a password meeting policy, when I submit,
  then an account is created and I receive confirmation.
- Password policy is enforced (exact rules: Phase 7); a non-conforming password is rejected
  with a clear message.
- Registering with an already-used email is rejected without revealing whether the email
  exists (anti-enumeration; detail: Phase 7).
- The password is never stored in plaintext (hashing detail: Phase 7).
- Invalid email format is rejected with a validation error.

### A2 — Log in with email/password
*As a user, I want to log in with email and password and receive a session token, so that I
can access my documents.*
**Acceptance criteria:**
- Valid credentials return a JWT usable for subsequent authenticated requests.
- Invalid credentials are rejected with a generic error (no hint whether the email exists
  or the password was wrong).
- No session state is kept server-side for this flow (stateless JWT).

### A3 — Log in with Google
*As a user, I want to log in with Google, so that I can skip password management.*
**Acceptance criteria:**
- A successful Google OAuth2 flow results in an authenticated session (JWT) for the user.
- A brand-new Google identity with no matching verified local email creates a new `User`.
- Linking behavior to an existing account: see A4.

### A4 — Safe account linking
*As a user with both a password and Google identity on the same email, I want them to
resolve to one account, so that my data isn't split.*
**Acceptance criteria:**
- Auto-link occurs only when the email is verified on both the OAuth provider's assertion
  and the local account.
- If the local email is not verified, linking requires an explicit, authenticated step
  (not automatic) — prevents account-takeover via email guessing.
- After linking, the user can log in via either method and reaches the same account/data.

### A5 — Log out
*As a user, I want to log out, so that my session ends.*
**Acceptance criteria:**
- Client discards the token; documented client-side behavior (JWTs are stateless, so
  server-side invalidation strategy, if any, is decided in Phase 7).

### A6 — Verify email
*As a user, I want to verify my email, so that account linking is safe.*
**Acceptance criteria:**
- A verification link/token is sent on registration.
- The account is marked verified only after the link is used; unverified accounts can still
  log in but cannot auto-link a second identity (see A4).

## Epic B — Document management

*A "document" = saved extracted text + metadata. Never an image (ADR-0001).*

| # | Story | Priority |
|---|---|---|
| B1 | Save extracted text with a title | Must |
| B2 | View a saved document | Must |
| B3 | List documents with pagination + sorting | Must |
| B4 | Edit a document's text/title/tags/folder | Should |
| B5 | Delete a document | Must |
| B6 | Search documents | Should |
| B7 | Filter by folder / tags / date | Should |

### B1 — Save a document
*As a user, I want to save extracted text with a title, so that I can keep it.*
**Acceptance criteria:**
- Given I'm authenticated, when I submit extracted text (+ optional title, tags, folder),
  then a document is created, owned by me.
- The request contains no image data; the endpoint accepts text and metadata only.
- Extracted text is required and non-empty; empty text is rejected with a validation error.
- Title is optional. If omitted, one is auto-derived (first ~40 characters of the text, or a
  timestamp if the text is empty-after-trim). *(Decision 1 — default applied.)*
- Client-computed metadata (confidence, page count, timestamp) is stored if provided.
- The document is visible only to its owner.

### B2 — View a document
*As a user, I want to view a saved document, so that I can read it later.*
**Acceptance criteria:**
- Returns the document only if it belongs to the requesting user; otherwise not found
  (never reveal existence of another user's resource).

### B3 — List documents
*As a user, I want to list my documents with pagination and sorting, so that I can browse
large histories.*
**Acceptance criteria:**
- Results are scoped to the requesting user only.
- Supports pagination (page/size) and sorting (e.g., by created date, title).
- Sensible defaults if no pagination/sort params are given.

### B4 — Edit a document
*As a user, I want to edit a document's text/title/tags/folder, so that I can correct or
organize it.*
**Acceptance criteria:**
- Only the owner can edit; edits are rejected for non-owned documents.
- Same validation rules as create (e.g., text cannot be edited to empty).

### B5 — Delete a document
*As a user, I want to delete a document, so that I can remove what I no longer want.*
**Acceptance criteria:**
- Deletion is a **hard delete** — the record is permanently removed. *(Decision 5 — default
  applied; see ADR-0004 for rationale.)*
- Only the owner can delete; deleting a non-owned document is rejected.

### B6 — Search documents
*As a user, I want to search my documents, so that I can find one fast.*
**Acceptance criteria:**
- Search matches against title, tags, and extracted text. *(Decision 3 — default applied.)*
- Search is scoped to the requesting user only.
- An empty query returns a validation error or the unfiltered list (defined at API design,
  Phase 6).

### B7 — Filter documents
*As a user, I want to filter by folder, tags, or date, so that I can narrow results.*
**Acceptance criteria:**
- Filters are combinable with search and pagination.
- An empty result set returns an empty list, not an error.

## Epic C — Folder management

| # | Story | Priority |
|---|---|---|
| C1 | Create a folder | Must |
| C2 | List folders | Must |
| C3 | Rename a folder | Should |
| C4 | Delete a folder | Should |
| C5 | Move a document between folders | Should |

### C1 — Create a folder
*As a user, I want to create a folder, so that I can group documents.*
**Acceptance criteria:**
- Folder name is required, non-empty.
- Folder name is unique per user, case-insensitive ("Work" and "work" collide).
  *(Decision 2 — default applied.)*
- Duplicate name for the same user is rejected with a clear validation error.

### C2 — List folders
*As a user, I want to list my folders, so that I can navigate.*
**Acceptance criteria:**
- Scoped to the requesting user only.

### C3 — Rename a folder
**Acceptance criteria:**
- Same uniqueness rule as creation (Decision 2) applies to the new name.

### C4 — Delete a folder
*As a user, I want to delete a folder, so that I can remove one I no longer need.*
**Acceptance criteria:**
- Documents inside the deleted folder are **not deleted**. Their folder reference is set to
  null (moved to "Uncategorized"). *(Decision 4 — default applied: set-null.)*
- A document may legally have no folder (folder is optional/nullable).

### C5 — Move a document between folders
**Acceptance criteria:**
- Setting a document's folder to another of the user's own folders succeeds; setting it to
  a folder owned by someone else is rejected.

## Epic D — Profile & System

| # | Story | Priority |
|---|---|---|
| D1 | View profile | Should |
| D2 | Update display name / preferences | Could |
| D3 | Audit logging of security-relevant events | Should |

### D1 — View profile
**Acceptance criteria:** Returns the authenticated user's own account info only.

### D2 — Update profile
**Acceptance criteria:** Only mutable fields (e.g., display name) can be changed; email
changes (if ever supported) would re-trigger verification — out of scope for MVP.

### D3 — Audit logging
*As the system, I want to record security-relevant events (login, save, delete), so that
there's an audit trail.*
**Acceptance criteria:**
- Login (success/failure), document save, and document delete are recorded with user id and
  timestamp.
- Audit records are never exposed through a public API in MVP (internal/ops use only).

## Won't — this release (tracked for Phase 16)

Sharing/collaboration, export, rate limiting, admin panel, tag-management UI, multi-language
OCR tuning.

## Edge-case decisions applied (defaults — override anytime)

| # | Decision | Default applied |
|---|---|---|
| 1 | Save with no title | Auto-derive from text / timestamp |
| 2 | Folder name uniqueness | Unique per user, case-insensitive |
| 3 | What search covers | Title + tags + extracted text |
| 4 | Delete folder with documents | Set-null → "Uncategorized" |
| 5 | Delete semantics | Hard delete (see ADR-0004) |
