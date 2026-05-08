# PANL compatibility and versioning

PANL is the public name for what shipped under the internal codename
**KPOW** during early development. The on-disk format is the same;
the public surface (format name, file extension, extension
namespaces, profile names) was renamed.

This document defines the versioning model and the rules for reading
legacy `.kpow` files and writing forward-compatible PANL files.

## Versioning

The file header carries `version_major` (uint16) and `version_minor`
(uint16). PANL is currently `version_major = 1`, `version_minor = 0`.

Compatibility rules:

- A reader rejecting `version_major != 1` is correct for this spec.
- A reader MUST tolerate higher `version_minor` values within a
  shared major. Minor bumps are restricted to additive,
  backward-compatible changes (new optional chunk types, new manifest
  features that aren't in `requiredFeatures`, new official extension
  namespaces).
- A reader that doesn't recognise a value in
  `manifest.requiredFeatures` MUST refuse the file rather than render
  a silently incorrect view.

Major-version bumps are reserved for incompatible changes. PANL v1 is
the first stable release; there is no v2 yet.

## File extension policy

- New writers MUST emit `.panl`.
- Readers SHOULD accept both `.panl` and `.kpow` files. The file
  extension is informational — the format identity comes from the
  header magic and the manifest's `format` field.

## Magic byte sequences

For v1, writers continue to emit:

- Header magic: `KPOW\r\n\x1A\n` (8 bytes).
- Footer magic: `KPOWend\n` (8 bytes).

These are kept opaque for v1 — the magic identifies the on-disk
container format, not the public name. Treating the magic as opaque
means existing readers (including pre-rename builds and third-party
implementations) keep working without changes.

A future v2 MAY rotate the magic to `PANL\r\n\x1A\n` /
`PANLend\n`. Readers conformant to v1 are not required to handle
that.

## Manifest `format` field

The manifest's `format` string is the canonical, human-readable
declaration of the format name.

- New writers MUST emit `"PANL"`.
- Readers MUST accept both `"PANL"` and `"KPOW"` and treat them as
  equivalent.
- A validator SHOULD warn (informational) when it sees `"KPOW"` in a
  newly-written file (i.e. one whose `org.panl.debug` records a
  recent converter run); operationally, the warning surfaces stale
  writers.

## Extension namespaces

Official extensions moved from `org.kpow.*` to `org.panl.*`. Both
namespaces refer to the same logical extensions:

| Legacy namespace               | Current namespace              |
|--------------------------------|--------------------------------|
| `org.kpow.metadata`            | `org.panl.metadata`            |
| `org.kpow.presentation`        | `org.panl.presentation`        |
| `org.kpow.preferences`         | `org.panl.preferences`         |
| `org.kpow.panels`              | `org.panl.panels`              |
| *(none)*                       | `org.panl.bookmarks` *(new in PANL)* |
| `org.kpow.page-roles`          | `org.panl.page-roles`          |
| `org.kpow.debug`               | `org.panl.debug`               |
| `org.kpow.panel-detections` *(deprecated)* | folded into `org.panl.debug` (`category="panel-detection"`) |

Reader rules:

1. Readers MUST accept both namespaces. When both are present in the
   same file, `org.panl.*` takes precedence.
2. Readers MAY merge `org.kpow.*` content into the `org.panl.*` view
   internally.
3. Apps SHOULD NOT delete `org.kpow.*` extensions when they don't
   recognise them — the same preservation rule as any unknown
   extension.

Writer rules:

1. New writers MUST emit `org.panl.*` for the official extensions
   they understand.
2. When writing a file that previously used `org.kpow.*`, writers
   SHOULD migrate the recognised keys to `org.panl.*` and remove the
   legacy duplicates. If migration would be ambiguous (e.g. both
   keys present with different content), writers MUST preserve the
   legacy value verbatim and log a warning to
   `extensions["org.panl.debug"]` with category `"compat-migration"`.
3. Migration MUST be lossless. Anything the migrator doesn't
   recognise stays under `org.kpow.*` (or moves to
   `legacy.unmigrated.*`) and is preserved.

## Legacy flat EDIT shape

Files written before the extension refactor used a flat shape with
`metadata`, `panels`, `pageRoles`, `cover`, `provenance`, `custom`
directly at the top of the EDIT JSON.

| Legacy field                    | Migrates to                                |
|---------------------------------|--------------------------------------------|
| `metadata`                      | `extensions["org.panl.metadata"]` (flat)   |
| `metadata.identity`             | `extensions["org.panl.metadata"].identifiers` |
| `metadata.custom["panel_flip"]` | `extensions["com.panel-flip"]`             |
| `cover`                         | `extensions["org.panl.presentation"]`      |
| `panels`                        | `extensions["org.panl.panels"]`            |
| `panelDetections`               | `extensions["org.panl.debug"].entries[]` (category `panel-detection`) |
| `pageRoles`                     | `extensions["org.panl.page-roles"]`        |
| `provenance`                    | `extensions["org.panl.debug"].entries[]` (category `conversion`) |
| `custom["panel_flip"]`          | `extensions["com.panel-flip"]`             |
| Anything else                   | `extensions["legacy.unmigrated"].data` (preserved) |

Migration is **lossless** — anything the migrator doesn't recognise
ends up under `legacy.unmigrated*` so future tooling can audit it.

`panl migrate-edit FILE` upgrades an old file to the current shape
in place.

## CUST chunks (`*.import:*`)

The legacy converter writes raw imported sidecar bytes as CUST
chunks (e.g. `kpow.import:ComicInfo.xml`,
`kpow.import:ComicMeta.json`). New writers SHOULD use the
`panl.import:` prefix, but readers MUST accept either prefix and
treat them as equivalent.

## Profile name aliases

Tooling accepts both the public profile names (`Recommended`,
`Fast`, `Compact`, `Archival`, `Original`) and the internal
engineering ids documented in [PROFILES.md](PROFILES.md). Internal
ids are kept for benchmark traceability and are not surfaced to
normal users.
