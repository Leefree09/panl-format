# PANL — open comic container format

PANL is an open, modular file format for comics. One `.panl` file
holds the pages, the metadata, the panel layouts, the thumbnails, and
the reading state — with explicit reading order, fast random access,
and an editable overlay that doesn't disturb the immutable image
package.

PANL is what shipped under the internal codename **KPOW** during early
development. Existing `.kpow` files remain readable — see
[spec/COMPATIBILITY.md](spec/COMPATIBILITY.md).

## Repo layout

```
panl-format/
  README.md                  ← this file
  LICENSE
  CHANGELOG.md

  spec/                      ← core on-disk format
    PANL.md                  ← top-level entry point
    CONTAINER.md             ← header, footer, directory
    CHUNKS.md                ← chunk types and ids
    STREAMABLE.md            ← BOOT chunk and front-loaded layout
    EDIT.md                  ← editable overlay container + extension rules
    PROFILES.md              ← user-facing conversion profiles
    VALIDATION.md            ← validator rules
    COMPATIBILITY.md         ← versioning and KPOW migration

  extensions/                ← extensions live here, not in core
    README.md
    official/                ← stable org.panl.* extensions
    draft/                   ← reserved namespaces, not yet stable
    private/                 ← guidance for app-private namespaces

  schemas/                   ← JSON Schemas
    panl-manifest.schema.json
    panl-edit.schema.json
    extensions/

  conformance/               ← reference test vectors
    README.md
    tests/
    expected/

  samples/                   ← example .panl files
    README.md
```

## Design philosophy

- **Small core, big extension surface.** Core PANL only defines what
  every reader has to agree on: file layout, manifest, page index,
  image chunks, the streamable bootstrap, and the editable overlay
  container. Everything mutable — metadata, panels, page roles,
  bookmarks, reading progress — is an *extension* under a namespaced
  key inside the EDIT overlay.
- **Apps shape the format around their needs.** Official extensions
  live under `org.panl.*`. Apps may define private extensions under
  reverse-domain namespaces (`com.panel-flip`, `dev.example.reader`).
  Unknown extensions MAY be ignored when reading and MUST be preserved
  when writing.
- **The image package is immutable.** Edits never rewrite page bytes.
  All mutable state lives in the EDIT overlay, which is rewritten in
  place inside its reservation.
- **Readers degrade gracefully.** A reader from before an extension
  existed still opens the file and renders pages. Required core
  features are gated by `manifest.requiredFeatures`; everything else
  is optional.

See [spec/PANL.md](spec/PANL.md) for the full spec entry point.

## File extension and naming

- Format name: **PANL**
- File extension: **`.panl`**
- Tooling family: **PANL Toolbox**
  - CLI: **PANL Toolbox Terminal**
  - GUI: **PANL Toolbox Graphic**

`KPOW` survives only as the internal/legacy codename and as a
read-side compatibility alias for `.kpow` files written before the
rename.

## Conversion profiles

Public PANL profiles use simple, user-facing names:

| Profile       | Use case                                                    |
|---------------|-------------------------------------------------------------|
| `Recommended` | Default. Tiled AVIF, decode-only candidate selection, adaptive risky-tile audit, streamable layout, EDIT/extensions. |
| `Fast`        | Quicker conversion. Lower assurance — good for previews.    |
| `Compact`     | Smaller files; more aggressive visually-lossless settings.  |
| `Archival`    | Strictest audit, slowest, maximum confidence.               |
| `Original`    | Preserve original image bytes verbatim inside the container. Useful when the source has little compression headroom. |

Internal engineering profile IDs (e.g. `reader-balanced-tiled-smart`)
still exist in the reference implementation and accept the public
names as aliases. Normal users only see the public names.

See [spec/PROFILES.md](spec/PROFILES.md) for the full mapping.

## Status

PANL is approaching its v1.0 spec freeze. The format is the same
on-disk shape as KPOW v1; what changed is the public name, the
extension namespace (`org.kpow.*` → `org.panl.*`), and the user-facing
profile names. See [CHANGELOG.md](CHANGELOG.md).

## Tooling

The reference implementation and tooling live in separate repos:

- **`panl-toolbox-terminal`** — Python CLI (`panl convert`, `panl
  validate`, `panl info`, …). Successor to the `kpow` CLI.
- **`panl-toolbox-graphic`** — desktop GUI for browsing, converting,
  and editing `.panl` files.

Both consume the spec in this repo and the JSON Schemas under
[schemas/](schemas/).

## License

See [LICENSE](LICENSE).
