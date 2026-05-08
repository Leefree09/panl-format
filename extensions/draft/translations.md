# `org.panl.translations` *(draft)*

- **Namespace**: `org.panl.translations` *(reserved)*
- **Status**: draft — not yet specified. Don't ship under the
  official name. Use a private namespace
  (e.g. `com.example.translations`) until this draft is promoted.

## Intent

Alternate-language metadata and per-panel translation overlays.
Lets a single file carry localized titles, descriptions, character
names, and (optionally) translated dialogue without forking into
per-language files.

## Open design questions

- Scope: just metadata translations (title, description, characters),
  or also panel-text translations / overlay text? The latter is
  much bigger and overlaps with panel data + annotations.
- Panel text overlays would be a separate concern from
  bibliographic metadata translations. Likely two extensions:
  `org.panl.translations` for metadata, something like
  `org.panl.dialog-overlay` for panel text.
- Source of truth: language tags via BCP 47 (`en-US`, `ja`,
  `pt-BR`) or ISO 639-1 only? Probably BCP 47 for flexibility.
- Fallback chain: how does a reader pick which translation to show?
  Likely a preference order from the host app, falling back to the
  metadata extension's `language`.

## Interim guidance

Apps that need translation overlays today SHOULD use a reverse-
domain namespace and migrate once promoted.
