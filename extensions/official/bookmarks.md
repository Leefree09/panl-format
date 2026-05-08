# `org.panl.bookmarks`

- **Namespace**: `org.panl.bookmarks`
- **Status**: official
- **Version**: `schemaVersion = "1.0"`
- **Predecessor**: none (promoted from draft; private bookmark
  namespaces remain valid and should be preserved)

## Purpose

Per-comic, file-portable jump targets: classic bookmarks, chapter
markers, scene starts, or any user-defined bookmark type.

This extension is intentionally flexible. A bookmark's `kind` is
free-form so apps can support custom workflows without waiting for a
spec update. Use `kind: "chapter"` for chapter-style TOC entries.

## Required / optional

Optional. A reader that does not support this extension still opens
and renders the comic normally.

## Fields

```json
{
  "schemaVersion": "1.0",
  "entries": [
    {
      "id": "bm-cover",
      "kind": "bookmark",
      "label": "Cover",
      "target": {"page": 0}
    },
    {
      "id": "ch-001",
      "kind": "chapter",
      "label": "Chapter 1",
      "order": 10,
      "target": {"page": 12},
      "tags": ["toc", "story"]
    },
    {
      "id": "scene-12a",
      "kind": "scene-start",
      "label": "Train Fight",
      "parentId": "ch-001",
      "target": {"page": 18, "panelId": "p3"}
    }
  ],
  "custom": {}
}
```

### Top-level

| Field           | Type                 | Notes                                                  |
|-----------------|----------------------|--------------------------------------------------------|
| `schemaVersion` | string               | Required. `"1.0"`.                                     |
| `entries`       | array of bookmark    | Required. MAY be empty.                                |
| `custom`        | object               | Optional. App-specific extras.                         |

### Bookmark entry

| Field      | Type           | Notes                                                  |
|------------|----------------|--------------------------------------------------------|
| `id`       | string         | Required. Stable identifier within this file.          |
| `kind`     | string         | Required. Free-form bookmark type (`bookmark`, `chapter`, `scene-start`, etc.). |
| `label`    | string         | Optional. User-visible title.                          |
| `target`   | target object  | Required. Jump location.                               |
| `order`    | integer        | Optional. Explicit sort order hint.                    |
| `parentId` | string         | Optional. Parent entry id for hierarchy (e.g. scene under chapter). |
| `tags`     | array<string>  | Optional. Free-form classification tags.               |
| `custom`   | object         | Optional. App-specific extras.                         |

### Target object

| Field      | Type             | Notes                                                  |
|------------|------------------|--------------------------------------------------------|
| `page`     | integer          | Optional. Logical page index.                          |
| `panelId`  | string           | Optional. Panel identifier on `page`. If present, `page` SHOULD also be present. |
| `chunkId`  | string           | Optional. Direct chunk anchor when page index is unstable. |
| `locator`  | object           | Optional. App-defined anchor object (free-form).       |

At least one target anchor SHOULD be present (`page`, `chunkId`, or
`locator`).

## Validation rules

| Check                                                  | Severity |
|--------------------------------------------------------|----------|
| Payload not an object                                  | error    |
| `schemaVersion` missing                                | warning  |
| `entries` not an array                                 | error    |
| Entry missing `id`, `kind`, or `target`               | error    |
| Duplicate `id` values in `entries`                     | error    |
| `target` has none of `page`, `chunkId`, `locator`     | warning  |
| `panelId` present without `page`                       | warning  |
| `page` negative                                         | error    |
| `parentId` points to a missing `id`                    | warning  |
| Cyclic parent graph                                    | warning  |

## Preservation rules

- Writers MUST preserve unknown top-level keys round-trip.
- Writers MUST preserve unknown fields on each bookmark entry.
- Writers MUST preserve unknown fields inside `target` and `custom`.

## Compatibility notes

- This namespace was previously draft-only. Experimental payloads may
  exist under private namespaces like `com.example.bookmarks`.
- Readers MAY map known private bookmark namespaces into this view for
  convenience, but writers MUST preserve the original private payloads
  losslessly unless a user explicitly migrates.
