# `org.panl.annotations` *(draft)*

- **Namespace**: `org.panl.annotations` *(reserved)*
- **Status**: draft — not yet specified. Don't ship under the
  official name. Use a private namespace
  (e.g. `com.example.annotations`) until this draft is promoted.

## Intent

User annotations / highlights anchored to a specific page (or panel)
— text notes, highlights, ink scribbles, sticky-notes, voice
memos. Read-only by other apps unless they explicitly support
editing.

## Open design questions

- Anchor model: page-relative `(page, bbox)` vs `(page, panelId)`
  vs polygon vs free-pixel. Need both panel-aware and panel-blind
  apps to interoperate.
- Mutation model: list-replace vs entry-by-id editing vs CRDT.
  Multi-device sync probably wants per-entry ids and a
  last-writer-wins `updatedAtMs`.
- Privacy: should annotations support per-entry "private to this
  user" flags? PANL files are typically shared as whole units, so
  this matters.
- Scope of supported content: text only? markdown? embedded blob
  (audio note)? If blobs, do they live as `CUST` chunks or inline
  base64 in the extension?

## Interim guidance

Apps that need annotations today SHOULD use a reverse-domain
namespace (`com.<app>.annotations`) and migrate to
`org.panl.annotations` once this draft is promoted. The migration
path will be documented in
[../../spec/COMPATIBILITY.md](../../spec/COMPATIBILITY.md) at
promotion time.
