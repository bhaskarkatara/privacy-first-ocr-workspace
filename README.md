# Privacy First OCR Workspace

Extract text from images (receipts, notes, screenshots, scans) **without ever handing the
raw image to a server**. OCR runs entirely in the browser via Tesseract.js. An authenticated
cloud account persists only the text and metadata the user *explicitly* chooses to save.

The architectural heart of this project is a **trust boundary**: the backend exposes
**no endpoint that accepts an image**. There is nothing on the server to receive, store,
or leak an image. That claim is enforced by the API surface itself, not by policy.

## Status

In development. Being built phase-by-phase (requirements → design → walking skeleton →
thicken each layer). See `docs/` for decisions made so far.

## Tech stack

- **Frontend:** React, TypeScript, Tesseract.js
- **Backend:** Java 21, Spring Boot, Spring Security, Spring Data JPA / Hibernate
- **Data:** PostgreSQL, Redis
- **Auth:** Email/password + JWT, and Google OAuth2
- **Docs:** OpenAPI / Swagger
- **Infra:** Docker, GitHub Actions CI/CD, AWS (free tier)

## Repository layout

```
privacy-first-ocr-workspace/
├── README.md                    # this file
├── docs/
│   ├── requirements.md          # Phase 0: problem, actors, trust boundary, scope
│   └── adr/                     # Architecture Decision Records
│       ├── 0001-client-side-ocr.md
│       ├── 0002-user-identity-model.md
│       └── 0003-single-user-workspaces.md
├── backend/                     # Spring Boot app  (internal structure: Phase 9/10)
├── frontend/                    # React/TS app     (internal structure: Phase 8)
├── docker-compose.yml           # local Postgres + Redis (added in Phase 13)
└── .github/workflows/           # CI/CD pipeline   (added in Phase 14)
```

`backend/` and `frontend/` are intentionally absent until the walking-skeleton phase
(Phase 11). They will be created when we build the first end-to-end slice.

## Architecture Decision Records (ADRs)

An ADR captures **why** a significant decision was made, so future-you (and interviewers)
can see the reasoning, not just the result. Each records context, the decision, and its
consequences. See `docs/adr/`.
