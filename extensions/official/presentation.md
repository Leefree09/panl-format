# `org.panl.presentation`

- **Namespace**: `org.panl.presentation`
- **Status**: official
- **Version**: `schemaVersion = "1.0"`
- **Predecessor**: `org.kpow.presentation` (read-side alias)

## Purpose

How the comic appears in a library / UI: which page is the cover,
which thumbnail to show in a grid, what title to display and how to
sort. This is presentation state, not bibliographic metadata.

## Required / optional

Optional. A reader without this extension falls back to
`manifest.coverPage` and the metadata title.

## Fields

```json
{
  "schemaVersion": "1.0",
  "coverPage": 0,
  "coverThumbnailChunkId": "thumb_cover",
  "displayTitle": "Hawkeye #1",
  "sortTitle": "Hawkeye 001",
  "seriesSortTitle": "Hawkeye",
  "collectionTitle": null,
  "preferredMetadataExtension": "org.panl.metadata",
  "custom": {}
}
```

| Field                        | Type                | Notes                                                  |
|------------------------------|---------------------|--------------------------------------------------------|
| `schemaVersion`              | string              | Required. `"1.0"`.                                     |
| `coverPage`                  | integer             | Optional. Page index for the cover. Overrides `manifest.coverPage`. MUST be in range `[0, pageCount)`. |
| `coverThumbnailChunkId`      | string              | Optional. Chunk id of a thumbnail to use in library grids. |
| `displayTitle`               | string              | Optional. Title to show in UI.                         |
| `sortTitle`                  | string              | Optional. Title to use for alphabetical sort.          |
| `seriesSortTitle`            | string              | Optional. Series name for series-level sort.           |
| `collectionTitle`            | string \| null      | Optional. Containing collection / arc name.            |
| `preferredMetadataExtension` | string (namespace)  | Optional. Namespace of the metadata extension a reader should prefer when more than one is present (e.g. `org.panl.metadata` vs a private translation). |
| `custom`                     | object              | Optional. App-specific extras.                         |

## Validation rules

| Check                                                  | Severity |
|--------------------------------------------------------|----------|
| Payload not an object                                  | error    |
| `schemaVersion` missing                                | warning  |
| `coverPage` not an integer                             | error    |
| `coverPage` out of range                               | error    |
| `coverThumbnailChunkId` references a chunk that doesn't exist | error |
| `coverThumbnailChunkId` references a non-`THUM` chunk  | warning  |
| `preferredMetadataExtension` references an extension that isn't present | warning |

## Preservation rules

- Writers MUST preserve unknown top-level keys round-trip.
- Writers MUST preserve `custom` round-trip.

## Compatibility notes

- Legacy namespace: `org.kpow.presentation`.
- Pre-extension files used a top-level `cover` field with a similar
  shape. Migration is in
  [../../spec/COMPATIBILITY.md](../../spec/COMPATIBILITY.md).
