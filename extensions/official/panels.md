# `org.panl.panels`

- **Namespace**: `org.panl.panels`
- **Status**: official
- **Version**: `schemaVersion = "1.0"`
- **Predecessor**: `org.kpow.panels` (read-side alias)

## Purpose

Effective panel layouts for reading. One entry per page, with one or
more panel rectangles describing the on-page panel decomposition
that a panel-mode reader should walk through.

This is the panel data the **reader uses**. Detector debug output
(raw confidences, alternative candidates, fallback flags) belongs in
[debug.md](debug.md) under category `panel-detection`, not here.

## Required / optional

Optional. A reader without panel data falls back to a single
full-page panel per page. The full effective-panels authority order
is in [../../spec/EDIT.md](../../spec/EDIT.md#effective-panels).

## Fields

```json
{
  "schemaVersion": "1.0",
  "source": "auto",
  "pages": [
    {
      "page": 0,
      "readingDirection": "ltr",
      "panels": [
        {
          "id": "p0",
          "order": 0,
          "bbox": [0.0, 0.0, 1.0, 1.0],
          "polygon": [[0.0, 0.0], [1.0, 0.0], [1.0, 1.0], [0.0, 1.0]],
          "confidence": 1.0,
          "source": "auto"
        }
      ]
    }
  ],
  "custom": {}
}
```

### Top-level

| Field           | Type                  | Notes                                                  |
|-----------------|-----------------------|--------------------------------------------------------|
| `schemaVersion` | string                | Required. `"1.0"`.                                     |
| `source`        | enum string           | Required. One of `auto`, `manual`, `mixed`. Origin of the data overall. |
| `pages`         | array of page object  | Required. One entry per page that has panel data. Pages without panels MAY be omitted. |
| `custom`        | object                | Optional.                                              |

### Per-page

| Field              | Type                | Notes                                                  |
|--------------------|---------------------|--------------------------------------------------------|
| `page`             | integer             | Required. Index into the page index. MUST be in range. |
| `readingDirection` | enum string         | Optional. One of `ltr`, `rtl`, `ttb`, `webtoon`. Overrides the comic-level reading direction for this page only. |
| `panels`           | array of panel      | Required. At least one panel.                          |

### Per-panel

| Field         | Type                          | Notes                                                  |
|---------------|-------------------------------|--------------------------------------------------------|
| `id`          | string                        | Required. Stable identifier within the page.           |
| `order`       | integer                       | Required. 0-based reading order within the page.       |
| `bbox`        | `[left, top, width, height]`  | Required. Page-relative `[0, 1]` floats.               |
| `polygon`     | array of `[x, y]`             | Optional. Convex polygon, page-relative `[0, 1]`.      |
| `confidence`  | number                        | Optional. `[0, 1]`. Auto-detected panels SHOULD set this; manual panels MAY omit (treated as `1.0`). |
| `source`      | enum string                   | Optional. One of `auto`, `manual`. Defaults to the page's containing source. |

Coordinates are normalized to `[0.0, 1.0]` relative to the page,
top-left origin. `bbox` is `[left, top, width, height]`. `polygon`
vertices are listed clockwise.

## Validation rules

| Check                                                  | Severity |
|--------------------------------------------------------|----------|
| Payload not an object                                  | error    |
| `schemaVersion` missing                                | warning  |
| `source` not in `{auto, manual, mixed}`                | error    |
| `pages` not an array                                   | error    |
| Page `page` index out of range                         | error    |
| Page `panels` empty                                    | error    |
| Panel `bbox` not 4-tuple of numbers                    | error    |
| Panel `bbox` value outside `[0, 1]`                    | error    |
| Panel `polygon` vertex outside `[0, 1]` or wrong arity | error    |
| Panel `id` missing or duplicated within the page       | error    |
| Panel `order` missing or non-integer                   | error    |
| Panel `confidence` outside `[0, 1]`                    | warning  |

## Preservation rules

- Writers MUST preserve unknown top-level keys round-trip.
- Writers MUST preserve unknown per-page and per-panel keys.
- A writer that re-runs panel detection and replaces `pages` MUST
  log the replacement in `org.panl.debug` (category
  `panel-detection`) so the audit trail survives.

## Compatibility notes

- Legacy namespace: `org.kpow.panels`.
- `org.kpow.panel-detections` (now deprecated) used to hold raw
  detector output. New writers SHOULD log detector output to
  [debug.md](debug.md) under category `panel-detection` instead.
- Legacy `PANE` and `PDET` chunks are still readable; the
  effective-panels rules in [../../spec/EDIT.md](../../spec/EDIT.md)
  define precedence.
