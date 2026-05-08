# PANL conversion profiles

PANL files can be produced under different conversion strategies
depending on what the user values most: speed, file size, archival
confidence, or byte-perfect source preservation. This document
defines the **public, user-facing profile names** that PANL Toolbox
exposes.

Profiles affect how a file is *produced*; they do not affect the
on-disk format. A reader cannot tell which profile produced a file
just by reading it.

## Public profile names

| Profile       | Use case                                                                   |
|---------------|----------------------------------------------------------------------------|
| `Recommended` | Default. Best balance of size, speed, and visual quality.                  |
| `Fast`        | Quicker conversion, lower assurance. Good for previews and quick imports.  |
| `Compact`     | Smaller output, more aggressive visually-lossless settings.                |
| `Archival`    | Strictest audit, slowest, maximum visual confidence.                       |
| `Original`    | Preserve original image bytes verbatim inside the PANL container.          |

Profile names are case-insensitive in tooling but canonically
title-cased in docs.

## Recommended (default)

The flagship profile — what `panl convert` uses with no flags.

- **Tiling**: 512px tiles per page.
- **Codec candidates**: AVIF only (no palette / WebP / JXL by default).
- **AVIF settings**: `speed=6`, `cq-level=20`, 4:2:0 chroma, AV1
  internal autotiling.
- **Quality gate**: decode-only candidate selection (cheap).
- **Audit**: adaptive risky-tile visual audit with targeted repair.
- **Layout**: streamable (BOOT chunk).
- **Overlay**: EDIT with the standard PANL extensions
  (`org.panl.metadata`, `presentation`, `preferences`, `panels`,
  `bookmarks`, `page-roles`, `debug`).

## Fast

For workflows where the user wants a `.panl` quickly — preview
imports, scratch conversions, batch ingestion to be re-converted
later.

- Faster / lighter audit mode if retained.
- Higher candidate cq-level / fewer candidates.
- Clearly labeled in tooling output as **lower assurance** so the
  user knows this isn't the archive-grade profile.

## Compact

For libraries that prioritise on-disk size, when source compression
headroom is available and visually-lossless results have been
validated for the codec being used.

- More aggressive visually-lossless settings (lower cq-level, smaller
  tiles, palette/WebP-lossless candidates enabled).
- Slower than `Recommended`; smaller output.
- Quality gate still required to pass before commit.

## Archival

For long-term preservation where the operator is willing to spend a
lot of CPU time to be sure the result is faithful.

- Strictest audit — exhaustive ladder, deep quality gate.
- Slowest profile (typically 5–7× `Recommended` wall-clock).
- Maximum confidence; intended for one-time archival passes.

## Original

For sources where the user wants byte-for-byte source preservation
inside the PANL container — no re-encoding, no transcoding.

- Page bytes are stored verbatim.
- Useful for books with little compression headroom (already-
  optimised AVIF, hand-curated source).
- The PANL file is essentially a structured wrapper around the
  original images, plus metadata and panels.

## Internal profile IDs

The reference implementation has internal engineering names used in
benchmarks and code (`reader-balanced-tiled-smart`,
`reader-fast-tiled-smart`, `reader-balanced-tiled-smart-light`,
`reader-balanced-tiled-smart-deep`, `reader-audit-tiled-smart`,
`archive-smallest`, `preserve-original`). Tooling continues to
accept these as aliases for backward compatibility:

| Public name   | Internal alias(es)                                                                |
|---------------|------------------------------------------------------------------------------------|
| `Recommended` | `reader-balanced-tiled-smart`                                                     |
| `Fast`        | `reader-fast-tiled-smart`, `reader-fast-convert-tiled-smart`                      |
| `Compact`     | `reader-mixed-tiled-smart`, `reader-balanced-tiled-smart-light`                   |
| `Archival`    | `reader-audit-tiled-smart`, `reader-balanced-tiled-smart-deep`, `archive-smallest` |
| `Original`    | `preserve-original`                                                                |

Normal users should not see the internal names. Tooling SHOULD print
the public name in its strategy block and SHOULD record the public
name in `extensions["org.panl.debug"]` (category `"conversion"`).
The internal id MAY also be recorded under `data.internalProfileId`
for benchmark traceability.

## Profile-related provenance

The conversion entry written to `extensions["org.panl.debug"]` (or
the legacy `PROV` chunk) records:

```json
{
  "id": "dbg-1777346178252",
  "createdAtMs": 1777346178252,
  "source": "org.panl.converter",
  "category": "conversion",
  "level": "info",
  "summary": "Converted via Recommended profile.",
  "data": {
    "profile": "Recommended",
    "internalProfileId": "reader-balanced-tiled-smart",
    "appId": "panl-toolbox-terminal",
    "appVersion": "1.0.0",
    "parallelism": {"workers": 10, "avifInternalThreads": 1, "mode": "max"},
    "candidates": ["avif"],
    "qualityGate": {"method": "decode-only"}
  }
}
```

This makes it possible to tell apart files produced by different
profiles after the fact, even though the on-disk shape is identical
across them.
