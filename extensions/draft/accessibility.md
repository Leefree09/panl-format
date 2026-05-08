# `org.panl.accessibility` *(draft)*

- **Namespace**: `org.panl.accessibility` *(reserved)*
- **Status**: draft — not yet specified. Don't ship under the
  official name. Use a private namespace
  (e.g. `com.example.accessibility`) until this draft is promoted.

## Intent

Accessibility metadata and reading aids: alt text per page or per
panel, transcribed dialogue, panel-level reading order corrections,
hints for screen-reader-style narration, dyslexia-friendly font
overrides, high-contrast color hints.

## Open design questions

- Granularity: alt text at page level, panel level, or both? Both
  is probably correct — a screen reader needs panel-level for
  navigation, page-level for skimming.
- Relationship to translations: dialogue transcription overlaps with
  translation overlays. Probably distinct: translation = "what the
  dialogue says in another language", accessibility transcription =
  "what the dialogue says, structured for assistive tech".
- Generated vs hand-curated: ML-generated alt text needs a flag so
  readers can warn. Confidence per entry?
- Audio narration tracks: do they live as `CUST` chunks linked from
  this extension, or as a separate extension entirely?
- Color/contrast hints: for color-blind readers, panels relying on
  red/green distinction may need annotated alternate-cue text.

## Interim guidance

Apps that need accessibility data today SHOULD use a reverse-
domain namespace and migrate once promoted. Until promotion, prefer
documenting the data shape openly so other apps can interoperate.
