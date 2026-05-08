# `org.panl.page-roles`

- **Namespace**: `org.panl.page-roles`
- **Status**: official
- **Version**: `schemaVersion = "1.0"`
- **Predecessor**: `org.kpow.page-roles` (read-side alias)

## Purpose

Per-page classification. Lets readers skip ads / credits / watermarks
in continuous-read mode, present "story pages only" library
thumbnails, jump to backmatter, etc.

The page index already carries an inline `role` per page (see
[../../spec/CHUNKS.md](../../spec/CHUNKS.md#page-index-pind-id-page_index)).
This extension is the **mutable** override / refinement of that
inline data, and lives in the EDIT overlay so editors can change it
without rewriting the immutable page index.

## Required / optional

Optional. When this extension is present, its values take precedence
over the inline `pages[i].role` in the page index.

## Fields

```json
{
  "schemaVersion": "1.0",
  "roles": {
    "0": "cover",
    "1": "story",
    "2": "story",
    "52": "backmatter"
  },
  "custom": {}
}
```

| Field           | Type                          | Notes                                                  |
|-----------------|-------------------------------|--------------------------------------------------------|
| `schemaVersion` | string                        | Required. `"1.0"`.                                     |
| `roles`         | object: stringified-int → role | Required. Keys are stringified page indexes (`"0"`, `"1"`, …). Values are role strings. Pages without an explicit role inherit from the page index. |
| `custom`        | object                        | Optional.                                              |

Allowed role values: `cover`, `story`, `ad`, `credits`, `backmatter`,
`watermark`, `extra`, `unknown`.

## Validation rules

| Check                                                  | Severity |
|--------------------------------------------------------|----------|
| Payload not an object                                  | error    |
| `schemaVersion` missing                                | warning  |
| `roles` not an object                                  | error    |
| `roles` key not a stringified non-negative integer     | error    |
| `roles` key out of range (`>= pageCount`)              | error    |
| `roles` value not in the allowed set                   | warning  |

## Preservation rules

- Writers MUST preserve unknown top-level keys round-trip.

## Compatibility notes

- Legacy namespace: `org.kpow.page-roles`.
- The same data may also appear as `pages[i].role` in the page index
  chunk. The page index is immutable; this extension is the editable
  layer.
