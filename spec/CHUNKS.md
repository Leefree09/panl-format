# PANL chunks

Defines chunk type codes, stable chunk ids, encoding rules, and the
JSON schemas of the well-known core chunks. The on-disk envelope —
header, footer, directory — is in [CONTAINER.md](CONTAINER.md).

## Chunk types (4-byte ASCII codes)

| Code   | Meaning                                              |
|--------|------------------------------------------------------|
| `MANI` | Manifest JSON                                        |
| `PIND` | Page index JSON (defines reading order)              |
| `TIND` | Thumbnail index JSON                                 |
| `PAGE` | Page image bytes (codec recorded in PIND)            |
| `TILE` | Single tile of a tiled page (codec recorded in PIND) |
| `THUM` | Thumbnail image bytes (codec recorded in TIND)       |
| `CUST` | Custom namespaced data (id is `namespace:id`)        |
| `BOOT` | Bootstrap (front-loaded chunk pointer table)         |
| `EDIT` | Editable comic-intelligence overlay                  |
| `META` | *(legacy)* Normalized metadata JSON. New writers SHOULD prefer the EDIT extension `org.panl.metadata`. |
| `PROV` | *(legacy)* Provenance / import history JSON. New writers SHOULD log into `org.panl.debug` instead. |
| `PANE` | *(legacy)* Effective panel layouts JSON. New writers SHOULD prefer `org.panl.panels`. |
| `PDET` | *(legacy)* Raw / auto panel detections. Folded into `org.panl.debug` (category `panel-detection`). |
| `DDUP` | Optional dedup map (SHA-256 + length + ref count per chunk) |
| `DELT` | Optional lossless tile delta — reference TILE + correction (prototype) |
| `PALT` | Optional palette/indexed-lossless tile (smart-tile profile) |

**Readers MUST tolerate unknown chunk types.** Listing them via
`list_chunks()` is fine; aborting reading is not. Strict-mode
validation MAY warn.

For details of the optional features (DDUP, DELT, PALT), see the
reference implementation; they are part of v1 but off by default and
documented in their own implementation notes.

## Stable chunk ids

Well-known chunks use these ids:

| Chunk type | Id                     |
|------------|------------------------|
| `MANI`     | `manifest`             |
| `PIND`     | `page_index`           |
| `TIND`     | `thumb_index`          |
| `EDIT`     | `edit_overlay`         |
| `BOOT`     | `bootstrap`            |
| `META`     | `metadata` *(legacy)*  |
| `PROV`     | `provenance` *(legacy)* |
| `PANE`     | `panels` *(legacy)*    |
| `PDET`     | `panel_detections` *(legacy)* |

Page and thumbnail chunks use:

- `page_NNNN` — 4-digit zero-padded, e.g. `page_0000`
- `thumb_NNNN`, e.g. `thumb_0000`
- For tiled pages: `page_NNNN_tile_CCCC_RRRR` (column, row)

Custom chunks use `namespace:id`, e.g. `com.example.future:demo`.

## Encoding / compression

For v1 the only valid `encoding` value is `0 = raw`. JSON chunks are
stored as compact UTF-8 JSON (no whitespace). Image chunks are stored
verbatim — the container does not decode or transcode them. The image
codec is recorded per-page in the page index (and per-thumb in the
thumbnail index).

Supported codec labels: `jpeg`, `png`, `webp`, `avif`, `jxl`,
`unknown`.

## Manifest (`MANI`, id `manifest`)

```json
{
  "format": "PANL",
  "version": {"major": 1, "minor": 0},
  "pageCount": 2,
  "coverPage": 0,
  "defaultReadingDirection": "ltr",
  "features": {
    "metadata": true, "thumbnails": true, "panels": true,
    "tiling": false, "dedup": false,
    "crossPagePrediction": false, "textLayer": false
  },
  "requiredFeatures": []
}
```

Notes:

- `format` is `"PANL"` for new files. Readers MUST also accept the
  legacy value `"KPOW"` and treat it as equivalent. See
  [COMPATIBILITY.md](COMPATIBILITY.md).
- `requiredFeatures[]` lists feature names a reader MUST understand
  to safely render the file. A reader that doesn't understand any
  listed feature MUST refuse the file rather than render a silently
  incorrect view. Used to gate optional features that change rendered
  output (e.g. `paletteTile` for smart-tile palette chunks).

The full JSON Schema is at
[../schemas/panl-manifest.schema.json](../schemas/panl-manifest.schema.json).

## Page index (`PIND`, id `page_index`)

Pages MUST be in reading order. **Readers do not infer order from
filenames or chunk ids** — `pages[i].chunkId` is the only
authoritative pointer.

```json
{
  "schemaVersion": "1.0",
  "pages": [
    {
      "index": 0,
      "chunkId": "page_0000",
      "codec": "avif",
      "width": 1988,
      "height": 3056,
      "role": "cover",
      "thumbnailChunkId": "thumb_0000",
      "tiles": null
    }
  ]
}
```

Tiled pages declare `tiles` instead of (or alongside) the page-level
`chunkId`:

```json
{
  "index": 1,
  "tiles": [
    {
      "col": 0, "row": 0,
      "x": 0, "y": 0, "width": 512, "height": 512,
      "chunkId": "page_0001_tile_0000_0000",
      "codec": "avif",
      "representation": "encoded"
    }
  ]
}
```

Allowed `role` values: `cover`, `story`, `ad`, `credits`,
`backmatter`, `watermark`, `extra`, `unknown`.

## Thumbnail index (`TIND`, id `thumb_index`)

Each entry's `page` MUST be a valid index into the page index, and
`chunkId` MUST resolve to a `THUM` chunk in the directory.

```json
{
  "schemaVersion": "1.0",
  "thumbnails": [
    {"page": 0, "chunkId": "thumb_0000", "codec": "webp",
     "width": 240, "height": 360}
  ]
}
```

## Required chunks

For a v1 file to validate:

- `MANI` (`manifest`) MUST be present.
- `PIND` (`page_index`) MUST be present.
- At least one `PAGE` (or `TILE`-bearing) chunk MUST be referenced
  via the page index.
- For each entry in the page index, the referenced `chunkId`(s) MUST
  exist and point to a chunk of the expected type.
- For each thumbnail entry (if `thumb_index` is present), the
  referenced `chunkId` MUST exist (preferably typed `THUM`).
- Manifest `pageCount` MUST equal the number of pages in
  `page_index`.

See [VALIDATION.md](VALIDATION.md) for the full validation table.
