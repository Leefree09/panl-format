# Changelog

All notable changes to the PANL specification are documented here.
The on-disk format is versioned via the `version_major` /
`version_minor` fields in the file header; this changelog tracks the
spec text alongside it.

## [Unreleased] — public PANL rename

The format previously shipped under the internal codename **KPOW**
v1. The on-disk shape is unchanged. What changed:

### Renamed

- Format name: `KPOW` → `PANL`.
- File extension: `.kpow` → `.panl`.
- Tooling family: introduced **PANL Toolbox** with two members —
  PANL Toolbox Terminal (CLI) and PANL Toolbox Graphic (GUI).
- Manifest `format` field: writers now emit `"PANL"`. Readers MUST
  also accept `"KPOW"` as a legacy alias.
- File magic: writers continue to emit the v1 magic byte sequence
  (`KPOW\r\n\x1A\n`) so existing readers keep working. The PANL spec
  treats this magic as opaque; a future v2 may rotate it.
- Footer magic: `KPOWend\n` is still authoritative; treat as opaque.
- Official extension namespace: `org.kpow.*` → `org.panl.*`.
- User-facing conversion profiles renamed:
  - `reader-balanced-tiled-smart` (flagship) → `Recommended`
  - `reader-fast-tiled-smart`               → `Fast`
  - `reader-mixed-tiled-smart` /
    `reader-balanced-tiled-smart-light`     → `Compact`
  - `reader-audit-tiled-smart` /
    `archive-smallest`                      → `Archival`
  - `preserve-original`                     → `Original`
  - Internal engineering IDs are kept as aliases for tooling and
    benchmarks.

### Reorganised

- The single monolithic `FORMAT.md` was split into focused docs under
  [spec/](spec/): container, chunks, streamable, edit, profiles,
  validation, compatibility.
- Extension docs were extracted out of `EDIT.md` into per-extension
  files under [extensions/official/](extensions/official/) (and
  [extensions/draft/](extensions/draft/) for reserved-only).
- JSON Schemas are now under [schemas/](schemas/), with one schema
  per official extension under [schemas/extensions/](schemas/extensions/).

### Extensions

- Promoted `org.panl.bookmarks` from draft to official.
- Added official extension spec:
  [extensions/official/bookmarks.md](extensions/official/bookmarks.md).
- Added JSON Schema:
  [schemas/extensions/org.panl.bookmarks.schema.json](schemas/extensions/org.panl.bookmarks.schema.json).

### Compatibility

- Existing `.kpow` files remain readable. PANL readers detect the
  legacy `KPOW` value in `manifest.format` and the `org.kpow.*`
  extension keys, and behave identically.
- Writers default to `.panl` output and `org.panl.*` extension keys.
  When migrating a legacy file, writers SHOULD rewrite recognised
  `org.kpow.*` keys to their `org.panl.*` equivalents and record the
  migration in `extensions["org.panl.debug"]` (category
  `"compat-migration"`).

## [v1.0] — KPOW v1 reference implementation

The first stable on-disk format — see the spec docs in this repo for
the authoritative reference. Highlights:

- Header + footer redundancy with independent CRCs.
- Per-chunk CRC-32; deep validation re-reads every chunk.
- Streamable layout (`FLAG_STREAMABLE`, BOOT chunk).
- Editable overlay (EDIT chunk) with namespaced extensions.
- Optional features: lossless dedup (DDUP), tile delta (DELT),
  smart-tile palette (PALT).
- Reference profiles for reading-balanced, archival, and
  preserve-original conversion.
