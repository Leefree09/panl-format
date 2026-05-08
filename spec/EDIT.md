# PANL EDIT overlay

Defines the **EDIT chunk** — PANL's editable overlay container — and
the rules every extension must follow. Per-extension field
definitions are not in this document; they live under
[../extensions/official/](../extensions/official/) and
[../extensions/draft/](../extensions/draft/).

## What EDIT is for

PANL files split their state into two regions:

- **Immutable image package** — header, BOOT, manifest, page index,
  page / tile chunks, cover thumbnail, directory, footer. Edits
  never touch these bytes.
- **Editable overlay** (this section) — a single `EDIT` chunk that
  holds the file's mutable comic intelligence: metadata, panel
  layouts, page roles, app state, diagnostics.

```
        ┌──────────────────────────────┐
        │  PANL Core (immutable)       │
        │  — header                    │
        │  — manifest                  │
        │  — page / tile index         │
        │  — image chunks              │
        │  — streamable BOOT           │
        │  — directory + footer        │
        ├──────────────────────────────┤
        │  EDIT overlay (mutable)      │
        │  ┌─ extensions ────────────┐ │
        │  │  org.panl.metadata      │ │
        │  │  org.panl.presentation  │ │
        │  │  org.panl.preferences   │ │
        │  │  org.panl.panels        │ │
        │  │  org.panl.bookmarks     │ │
        │  │  org.panl.page-roles    │ │
        │  │  org.panl.debug         │ │  ← shared diagnostics
        │  │  com.<your-app>         │ │
        │  └─────────────────────────┘ │
        └──────────────────────────────┘
```

The overlay is allocated at conversion time with a fixed *capacity*
and rewritten in place when the new payload still fits.

A v1+ converter writes `EDIT` by default with 1 MB of reserved
capacity (4 MB for books with large panel data; configurable via
`--edit-overlay-size`). Files written with `--no-edit-overlay` keep
metadata in the legacy `META` / `PANE` / `PDET` chunks instead.

## EDIT payload binary header

```
4 bytes   "KEDT"          payload magic
2 bytes   overlay_format_version (uint16, currently 1)
2 bytes   reserved (uint16, 0)
4 bytes   used_length (uint32) — bytes of JSON that follow
4 bytes   payload_crc32 (uint32) — CRC over header [0..16) (with crc=0)
                                    + JSON bytes
N bytes   json_bytes (compact UTF-8 JSON, length = used_length)
... padding (zeros) up to capacity (chunk-directory `length`)
```

`used_length` is authoritative — readers MUST trust it over the
chunk's outer `length` (the on-disk reservation).

The EDIT chunk's directory entry uses `SKIP_CRC_SENTINEL` (`0`); the
chunk validates itself via the internal `payload_crc32`. See
[CONTAINER.md](CONTAINER.md#crcs).

## EDIT JSON container shape

```json
{
  "schemaVersion": "1.0",
  "updatedAtMs": 1777346178252,
  "extensions": {
    "org.panl.metadata":     {...},
    "org.panl.presentation": {...},
    "org.panl.preferences":  {...},
    "org.panl.panels":       {...},
    "org.panl.bookmarks":    {...},
    "org.panl.page-roles":   {...},
    "org.panl.debug":        {...},
    "com.panel-flip":        {...}
  }
}
```

Top-level fields are reserved for **container metadata only**:
`schemaVersion`, `updatedAtMs`, `extensions`, and (future)
`extensionManifest`. Everything else lives inside an extension.

The container schema is
[../schemas/panl-edit.schema.json](../schemas/panl-edit.schema.json).

Required:
- `schemaVersion`.
- `extensions` (may be empty).

## Extension rules

These rules apply to every extension, official or private.

1. `EDIT.extensions` is an object keyed by namespace.
2. Extension keys MUST be namespaced.
3. Official PANL extensions use `org.panl.*`. They are catalogued in
   [../extensions/official/](../extensions/official/).
4. App / private extensions SHOULD use a reverse-domain namespace
   (`com.example.app`, `io.acme.viewer`, `dev.example.reader`). See
   [../extensions/private/README.md](../extensions/private/README.md).
5. **Unknown extensions MAY be ignored when reading. Unknown
   extensions MUST be preserved when writing.** This is the most
   important rule — apps that round-trip through PANL MUST NOT
   delete extensions they don't understand.
6. Each extension SHOULD include `schemaVersion`. Validators warn
   when missing; they don't fail.
7. Validators warn on generic / non-namespaced keys
   (`myExtension` → warning, `com.example.x` → fine).
8. Apps SHOULD use the official extension when a stable equivalent
   exists. Don't build `com.example.metadata` next to
   `org.panl.metadata`.
9. **Official extensions are optional unless a compatibility
   profile requires them.** A reader that doesn't recognise
   `org.panl.bookmarks` MUST still open the file.
10. Top-level `EDIT` fields are reserved for container metadata.
    Don't add new top-level keys; add an extension.

## Effective-value rules

When the same conceptual field could come from multiple places
(extensions, legacy fields, manifest defaults), readers consult
sources in this order. The official extensions referenced here are
defined under [../extensions/official/](../extensions/official/).

### Effective metadata
1. `extensions["org.panl.metadata"]`
2. legacy top-level `metadata` (pre-extension shape)
3. base `META` chunk
4. `null`

### Effective cover
1. `extensions["org.panl.presentation"].coverPage` /
   `coverThumbnailChunkId`
2. legacy top-level `cover`
3. `manifest.coverPage`
4. page `0`

### Effective reading direction
1. `extensions["org.panl.preferences"].readingDirection`
2. metadata hints
3. `manifest.defaultReadingDirection`
4. `"ltr"`

### Effective start page
1. `extensions["org.panl.preferences"].startPage`
2. first `story` page from `extensions["org.panl.page-roles"]`
3. `0`

### Effective bookmarks
1. `extensions["org.panl.bookmarks"]`
2. none

### Effective panels
1. extension named by
   `extensions["org.panl.preferences"].preferredPanelExtension`
2. `extensions["org.panl.panels"]`
3. legacy top-level `panels`
4. base `PANE` chunk
5. app detection / database
6. full-page panel (single panel covering the whole page)

## In-place updates

Writers MAY rewrite the EDIT chunk's bytes (header + JSON + zero
padding, exactly capacity bytes) in place when the new payload fits.
The directory entry's offset, length, encoding, and `SKIP_CRC`
sentinel stay unchanged; image / page / tile chunks aren't touched.

When the new payload exceeds capacity, the write MUST fail without
touching the file. Callers grow the overlay via a full repack, which
copies image bytes verbatim.

## Streamable + EDIT

For streamable files the EDIT chunk sits in the front-loaded region
(after BOOT and the metadata indexes, before bulk page data). The
BOOT JSON's `chunks.editOverlay` pointer carries an `offset`,
`length` (reservation), and informational `usedLength`. See
[STREAMABLE.md](STREAMABLE.md) for details.

## Validator rules for EDIT

| Check                                                            | Severity |
|------------------------------------------------------------------|----------|
| Bad magic / unsupported version                                  | error    |
| Internal payload CRC mismatch                                    | error    |
| `used_length` exceeds capacity                                   | error    |
| Missing `schemaVersion`                                          | error    |
| Extension key not namespaced                                     | warning  |
| Extension payload not an object                                  | error    |
| Streamable BOOT pointer disagrees with directory                 | error    |
| Overlay > 90% full                                               | warning  |
| Both EDIT and base META/PANE present (EDIT wins; META/PANE ignored) | warning |
| Per-extension validation errors (see each extension spec)        | varies   |

## Migration from KPOW (legacy `org.kpow.*`) and from the flat shape

Files written before the public PANL rename use `org.kpow.*`
extension keys. Files written before the extension refactor used a
flat shape with `metadata`, `panels`, `pageRoles`, `cover`,
`provenance`, `custom` directly at the top level.

The reader migration table is in [COMPATIBILITY.md](COMPATIBILITY.md).
The reference tools' `panl migrate-edit FILE` upgrades the EDIT
shape in place.

## Reader / writer responsibilities (summary)

- A reader looking up the comic's metadata, panel layouts, etc.
  MUST consult the effective-value rules above.
- A reader MUST tolerate unknown extensions, unknown chunk types,
  unknown `manifest.features.*` flags, and unknown `category` values
  in `org.panl.debug.entries[]`.
- A writer MUST round-trip unknown extensions untouched.
- A writer SHOULD update `updatedAtMs` whenever it rewrites the
  overlay JSON.
- A writer MUST set `extensions[ns].schemaVersion` for any extension
  it produces, when the extension's spec defines one.

Per-extension field shapes, required/optional rules, validation, and
preservation details are catalogued in
[../extensions/official/](../extensions/official/).
