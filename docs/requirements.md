# Requirements Analysis (Phase 0)

> Scope note: This document captures the *problem*, *actors*, *trust boundary*, and
> *scope skeleton*. Detailed user stories with acceptance criteria are Phase 2.
> Measurable non-functional targets (latency, throughput, etc.) are Phase 3.
> Keeping each phase in its lane is deliberate.

## Problem statement

People need to extract text from images (receipts, handwritten notes, screenshots,
scanned documents) without handing the raw image to a third-party server. Existing web
OCR tools upload the image to their backend. This product performs OCR entirely in the
browser and persists — to an authenticated cloud account — only the text and metadata
the user *explicitly* chooses to save.

## Actors (MVP)

- **Anonymous visitor** — can perform OCR in the browser; cannot save anything.
- **Authenticated user** — can save extracted text, organize documents into folders, and
  search / sort / filter / paginate their own content. Single-user workspace: a user only
  ever sees their own data.
- **System (audit)** — an internal, non-human actor that records security-relevant events
  (logins, saves, deletions).

Out of scope for MVP: admin role, collaboration/sharing (see Phase 16).

## Trust boundary (the architectural heart)

| Never crosses to the server           | Crosses only on explicit save                          |
| ------------------------------------- | ------------------------------------------------------ |
| Raw image bytes                       | Extracted text                                         |
| Thumbnails / any pixel-derived data   | User metadata: title, tags, folder                     |
| The image file itself, in any form    | Client-computed metadata: OCR confidence, page count, timestamp |
|                                       | Original file *name* — **opt-in only** (user data; slight privacy cost) |

**How the boundary is enforced (and how to prove it):** the server can't force a browser
to behave. It enforces the boundary by exposing **no endpoint that accepts an image**.
There is no image-upload route to attack, misconfigure, or leak from. To prove the claim
to a security reviewer, walk them through the API surface (OpenAPI spec) and show that no
route consumes image data.

## Scope

### MVP (Must-have skeleton)

- Authentication: email/password + JWT, and Google OAuth2
- Save extracted text (only on explicit user action)
- Folder management
- List / search / sort / filter / paginate the user's own documents
- View / edit / delete a saved document
- Basic user profile
- Audit logging
- API documentation (OpenAPI/Swagger)
- Input validation
- Global exception handling

### Later (tracked for Phase 16)

- Sharing / collaboration
- Export (e.g., to PDF / plain text)
- Rate limiting
- Tag management UI
- Multi-language OCR tuning
- Admin panel

## Explicit assumptions

- OCR quality and performance are the browser's responsibility (Tesseract.js). The backend
  never evaluates or reprocesses OCR output.
- Unsaved OCR results are ephemeral — lost when the page closes. This is the privacy
  tradeoff, and it will be surfaced clearly in the UI.
- One human = one `User`, potentially with multiple credentials (password and/or Google).
  See ADR-0002.

## Decisions logged

- **ADR-0001** — OCR is client-side; the backend has no image endpoint.
- **ADR-0002** — User identity is one `User` + one-or-more credential records; safe,
  email-verified account linking only.
- **ADR-0003** — Single-user workspaces for MVP.
