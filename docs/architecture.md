# Architecture Design (Phase 4)

> A system you can draw and defend. Captures the component map, the request lifecycle
> for a representative endpoint, and the client-side OCR flow. Package/folder structure
> lives in Phase 10; database schema lives in Phase 5.

## Component map with the trust boundary

![Component map with the trust boundary](diagrams/component-map.svg)

Four things worth noticing:

**The Tesseract.js worker sits above the boundary.** The entire OCR engine lives inside
the browser. A reviewer sees at a glance where images live and where they don't — the
privacy story becoming visual.

**Only JSON crosses the boundary.** The arrow from the API client is labelled — the only
payload that ever moves down is text plus metadata. There is no arrow for image data,
because there is no code to send it.

**The backend is four layers in strict order:** filter chain → controllers → services →
repositories. This is the Spring layered-architecture pattern that will show up in every
Java interview. Each layer's job is written on it. The Phase 10 folder structure will
mirror this exactly.

**Redis is connected by a dashed arrow** — visual grammar for "optional dependency." The
picture of NFR R2: the app must survive Redis being down. If Redis were solid, the app
would depend on it.

## Request lifecycle — saving a document

The component map shows what exists. This diagram shows what happens.

![Request lifecycle for saving a document](diagrams/save-request-lifecycle.svg)

Read top-to-bottom and notice the discipline of what each layer owns and deliberately
does not own:

**Filter chain.** First thing to touch the request. Cracks open the `Authorization`
header, verifies the JWT signature and expiry, resolves the user id, populates a
`SecurityContext`. If any of that fails, the request is rejected here with 401 — the
controller never runs. This is *authentication*.

**Controller.** Deliberately uninteresting. Job: receive the HTTP request, deserialize
JSON into a DTO, run declarative validation annotations (`@NotBlank`, `@Size`), map to
something the service accepts, call the service, wrap the response. **Zero business
logic.** If an `if` shows up in a controller, it belongs in the service.

**Service.** Where the product lives. Applies business rules from Phase 2: text
non-empty, auto-derive title if missing, resolve the folder reference, verify the folder
belongs to this user (this is the IDOR defense — SEC4), stamp timestamps, hand to the
repository. Wrapped in `@Transactional`; if the repository call fails, nothing is
written. The service knows nothing about HTTP.

**Repository.** Data access only. `documentRepository.save(entity)` — one line. No
validation, no ownership checks, no transaction management. If a "clever" repository
method starts creeping in with business logic, it moves up to the service.

The return trip is dashed because it is implicit: every layer's return value flows back
up the same chain, and the controller finally maps the saved entity to a response DTO
and returns a `201 Created`.

## Responsibility map

| Layer         | Owns                                                                                              | Does NOT own                                       |
| ------------- | ------------------------------------------------------------------------------------------------- | -------------------------------------------------- |
| Filter chain  | Authentication — validating the JWT, populating `SecurityContext`                                 | Business rules, per-resource authorization         |
| Controller    | HTTP concerns: routing, DTO shape, request validation annotations, response mapping               | Business rules, DB access, transactions            |
| Service       | Business rules, orchestration, transaction boundaries, **per-resource ownership checks**          | HTTP, SQL, response status codes                   |
| Repository    | Data access via JPA / Spring Data                                                                 | Business rules, transaction management             |
| Entity        | JPA persistence mapping; simple invariants only                                                   | Business rules that require external data          |
| DTO           | The shape data takes over the wire                                                                | Persistence, business logic                        |

The single line worth memorising: **authentication is "who are you"; authorization is
"are you allowed to touch this."** Filter chain does the first; service does the second.
Portfolio projects that fail SEC4 fail because they think authentication is enough.

## Client-side OCR flow

The last piece: how OCR happens without a server, and where the trust boundary is
actually crossed.

![Client-side OCR flow](diagrams/client-side-ocr-flow.svg)

**Why a Web Worker for OCR, not the main thread?** Tesseract.js is CPU-heavy. Running it
on the main JavaScript thread would freeze the UI — buttons unresponsive, spinners not
spinning. A Web Worker is a background thread the browser provides; the main thread
stays responsive while OCR grinds away.

**The fork is where the trust boundary earns its keep.** Copy dead-ends in gray — no
network call, nothing crosses, text is gone when the tab closes. Save is the only path
that ever generates an HTTPS request, and only text and metadata. "The backend has no
image endpoint" isn't just a sentence — the flow itself cannot draw an image-carrying
arrow, because no code produces one.

## Key architectural properties

For interview talking points, these are the properties this architecture buys:

- **Stateless backend.** No session state on the server; scaling horizontally is a
  configuration change, not a rewrite. Enabled by JWT (SEC1, S1).
- **Trust boundary via absence of endpoint.** ADR-0001 enforced by the API surface, not
  by policy.
- **Layered architecture with narrow responsibilities.** Controllers → services →
  repositories. Business logic can only live in one place.
- **Ownership checks in the service layer.** Every data-access path scoped to the
  authenticated user (SEC4).
- **Redis as optional performance layer.** Failure of Redis degrades performance, not
  correctness (R2).
- **All external I/O documented.** Only HTTPS-JSON in; PostgreSQL (required) and Redis
  (optional) out.
