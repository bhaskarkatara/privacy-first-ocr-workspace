# Product Vision (Phase 1)

## Vision statement

Privacy First OCR Workspace lets people turn images into organized, searchable text they can
keep — without ever entrusting the original image to a server. It gives the convenience of a
cloud workspace to the extracted *text*, while the sensitive source *image* never leaves the
user's device.

> This statement is intentionally technology-agnostic. It remains true regardless of the
> stack used to implement it — that is the test of a real vision.

## Value proposition

Cloud convenience for the text; zero cloud exposure for the image.

The value is not "OCR" — many tools do OCR. The value is getting a saved, searchable,
organized text history across devices for the *extracted text*, without paying the privacy
cost of uploading the *source image*.

## Primary persona

A **privacy-sensitive individual or professional** who regularly digitizes sensitive
documents — medical records, legal paperwork, financial statements, personal IDs — and
wants a searchable, organized history of the extracted text, but is unwilling or not
permitted to upload the source images to a third-party service.

## Secondary users (also served, not designed-for)

General users who simply want a fast OCR tool with a tidy, saved history.

## Differentiation

Most online OCR tools upload the image to their servers. This product runs OCR entirely in
the browser, so the image never leaves the device, and the backend exposes no endpoint that
can receive an image. Users still get a cloud-backed, cross-device, searchable text history.

## 30-second elevator pitch

> "I built a privacy-first OCR workspace. Most online OCR tools upload your image to their
> servers — a problem if you're scanning something sensitive like a medical bill or a
> contract. Mine runs OCR entirely in the browser, so the image never leaves your device.
> The backend only ever stores the extracted text and metadata you explicitly choose to
> save — so you still get a searchable, organized cloud history across devices, without the
> privacy cost. The interesting engineering challenge was designing the whole system around
> that trust boundary: the backend has no endpoint that can even receive an image."

## Anticipated interview question

**"Is the privacy angle a real user need, or just a technical exercise?"**
Someone digitizing a medical bill, legal document, passport, or client contract needs to
keep and organize the results without trusting a server with the sensitive original. That is
a concrete, nameable need — the project is built to serve it, not to showcase a technique.
