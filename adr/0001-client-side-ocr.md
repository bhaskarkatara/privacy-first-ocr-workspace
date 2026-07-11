# ADR-0001: OCR is performed client-side; the backend has no image endpoint

- **Status:** Accepted
- **Date:** 2026-07-11

## Context

The product's core promise is privacy: users extract text from images without trusting a
server with the raw image. Many OCR web tools upload the image for server-side processing,
which creates a data-handling and trust liability. We need a design where the privacy claim
is *structural*, not merely a policy statement.

## Decision

Perform all OCR in the browser using Tesseract.js. The backend exposes **no endpoint that
accepts image data** in any form (bytes, base64, thumbnail, or otherwise). The backend
persists only what the user explicitly chooses to save: extracted text and metadata.

## Consequences

**Positive**
- The privacy claim is enforced by the absence of an attack/leak surface, not by trust.
- No image storage, no image-processing infrastructure, no related compliance burden.
- The claim is demonstrable: the OpenAPI spec shows no image-accepting route.

**Negative / tradeoffs**
- OCR quality and speed depend on the client device; the backend cannot improve results.
- Heavy OCR runs on the user's CPU. Mitigated by running Tesseract.js in a Web Worker
  (Phase 8) so the UI stays responsive.
- Unsaved results are ephemeral. This is surfaced in the UI as an intentional tradeoff.

## Notes

The server cannot force a browser to behave. The boundary is enforced by never offering a
route that could receive an image — there is nothing to misconfigure or exploit.
