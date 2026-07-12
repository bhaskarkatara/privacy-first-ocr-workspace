# Non-functional Requirements (Phase 3)

> Rule for this document: every requirement is a **number** or a **yes/no**. If it can't be
> measured, it isn't here. Latency is measured server-side at the API layer.
>
> Posture: designed *as if* thousands of users will arrive; honestly stating what will
> actually be tested and verified on a solo, AWS-free-tier deployment.

## 1. Performance

| #  | Requirement                                                                       | Target                     |
| -- | --------------------------------------------------------------------------------- | -------------------------- |
| P1 | Auth endpoints (login/register) p95 latency                                       | < 500 ms                   |
| P2 | Read endpoints (list, get, search) p95 latency                                    | < 300 ms                   |
| P3 | Write endpoints (save, update, delete) p95 latency                                | < 400 ms                   |
| P4 | OCR extraction time (client-side, single A4 page, mid-tier laptop) — informational | < 5 seconds                |
| P5 | Server-side handling of a document save (payload up to 100 KB text) p95           | < 200 ms                   |

**Notes.** Login/register is intentionally slower because adaptive password hashing
(bcrypt/argon2 — Phase 7) is deliberately slow. Reads have the strictest budget because
that is what users perceive as "the app."

**How verified.** k6 / Apache Bench against a local Docker Compose stack; measured p95s
recorded in the repo README. No claimed number is unmeasured.

## 2. Scalability

| #  | Requirement                                                             | Target                                                                       |
| -- | ----------------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| S1 | Design supports horizontal scaling of the backend (stateless app tier)  | Yes — enforced by stateless JWT (no server-side session state)               |
| S2 | Handle 50 concurrent authenticated users with P1–P3 holding             | Yes, verified via load test                                                  |
| S3 | Realistic single-user workload: 1,000 saved documents per user          | Yes, without list/search regressing beyond P2                                |
| S4 | Explicitly **not** optimized for                                        | > 10k concurrent users, > 1M docs/user, real-time collaboration              |

**Notes.** S1 is the load-bearing scalability property: the *design* is horizontally
scalable even if only one instance is ever run. That is the interview story — "I could
put N instances behind a load balancer today because there is no server-side session state."

## 3. Security

| #     | Requirement                                                                                                                       | Target                                        |
| ----- | --------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------- |
| SEC1  | Passwords stored using an adaptive hash (bcrypt or argon2); never plaintext or reversible                                         | Yes                                           |
| SEC2  | All authenticated endpoints require a valid JWT; no endpoint accidentally public                                                  | Verified by integration tests (Phase 12)      |
| SEC3  | No endpoint accepts image data in any form (enforced by API surface)                                                              | Yes — ADR-0001 boundary                       |
| SEC4  | Every data-access query scoped by owner id (no IDOR)                                                                              | Yes, verified by cross-user access tests      |
| SEC5  | Login errors generic; do not reveal whether email exists or which field was wrong                                                 | Yes                                           |
| SEC6  | Registration does not reveal whether an email is already taken (anti-enumeration)                                                 | Yes                                           |
| SEC7  | Secrets (DB password, JWT signing key, Google OAuth client secret) never in source control                                        | Yes — env vars / secrets manager              |
| SEC8  | HTTPS required in deployed environments                                                                                           | Yes (terminated at load balancer in AWS)      |
| SEC9  | Input validation on every request DTO; rejected before service layer                                                              | Yes                                           |
| SEC10 | Dependency vulnerability scanning in CI                                                                                           | Yes (Phase 14)                                |

**Notes.** SEC4 is the requirement most portfolio projects fail. An explicit test that
authenticates as user A and confirms A cannot fetch user B's document is one of the
strongest single tests in the whole project.

## 4. Reliability & Availability

Honest posture on free-tier single-instance: no redundancy, no real uptime target.
Requirements here are about **graceful degradation and recovery**, not five-nines.

| #  | Requirement                                                                                     | Target |
| -- | ----------------------------------------------------------------------------------------------- | ------ |
| R1 | Application starts cleanly from cold in < 30 seconds                                            | Yes    |
| R2 | Failure of Redis does not cause request failures — app degrades to hitting Postgres directly    | Yes    |
| R3 | Database migrations are versioned and reversible where possible (Flyway)                        | Yes    |
| R4 | No data loss on graceful restart (DB is source of truth; no in-memory state)                    | Yes    |
| R5 | Explicitly **not** targeted                                                                     | HA, zero-downtime deploys, multi-region |

**Notes.** R2 is the important habit: Redis is a performance optimization, not a hard
dependency. Killing the Redis container must not 500 the app.

## 5. Maintainability

| #  | Requirement                                                                                         | Target                            |
| -- | --------------------------------------------------------------------------------------------------- | --------------------------------- |
| M1 | Layered architecture: controller → service → repository (Phase 9)                                   | Yes                               |
| M2 | Unit test coverage on service layer                                                                 | ≥ 70%                             |
| M3 | Integration tests cover every controller endpoint on happy path + one auth-failure path             | Yes                               |
| M4 | No business logic in controllers or repositories                                                    | Enforced by review + package layout |
| M5 | Consistent code style (formatter + linter run in CI)                                                | Yes                               |
| M6 | Every ADR-worthy decision recorded as an ADR                                                        | Yes                               |

**Notes.** M2 is 70%, not 100%. Chasing 100% produces bad tests. 70% on the service layer,
applied to *behavior* rather than lines, is the professional bar.

## 6. Observability

| #  | Requirement                                                                                                                       | Target |
| -- | --------------------------------------------------------------------------------------------------------------------------------- | ------ |
| O1 | Structured JSON logging (not plain-text `System.out.println`)                                                                     | Yes    |
| O2 | Every request logged with: method, path, status, latency, user id (if authenticated), correlation/request id                      | Yes    |
| O3 | Sensitive data (passwords, tokens, extracted text bodies) never appears in logs                                                   | Yes    |
| O4 | Health check endpoint (`/actuator/health`) available for load balancers                                                           | Yes    |
| O5 | Metrics endpoint via Micrometer exposing at least request count, latency histogram, error rate (`/actuator/metrics`)              | Yes    |

**Notes.** O3 extends the privacy story into operations. It would be embarrassing to
build a privacy-first product and dump extracted text into logs. Enforcing this in Phase 9
logging config is part of the product promise, not a stylistic choice.

## 7. Usability & Accessibility (backend-facing)

| #  | Requirement                                                                                                             | Target        |
| -- | ----------------------------------------------------------------------------------------------------------------------- | ------------- |
| U1 | Error responses follow a consistent JSON shape (RFC 7807 Problem Details or similar)                                    | Yes (Phase 6) |
| U2 | API documented via OpenAPI/Swagger, browsable at `/swagger-ui` in non-production                                        | Yes           |
| U3 | Pagination responses include total count and next-page indicator                                                        | Yes           |

## 8. Compliance & Data Handling

| #  | Requirement                                                                                                                       | Target                       |
| -- | --------------------------------------------------------------------------------------------------------------------------------- | ---------------------------- |
| C1 | User data deleted permanently on account deletion (no orphan records)                                                             | Yes (Phase 5 cascade design) |
| C2 | System never stores images, thumbnails, or pixel-derived data                                                                     | Yes — ADR-0001               |
| C3 | Extracted text treated as user-owned; not indexed, mined, or read for any purpose other than serving it back to the owner        | Yes — policy + no cross-user query paths |

## Bounded scope (what we are deliberately **not** doing)

- Not designed for > 10k concurrent users.
- Not targeting high availability, zero-downtime deploys, or multi-region.
- No rate limiting in MVP (tracked, Phase 16).
- Extracted text is not encrypted at the application layer at rest; relying on managed DB
  encryption at rest. An explicit ADR would be required to introduce app-level encryption
  later.

Naming what the design does not solve — and why — is a deliberate signal of maturity.
