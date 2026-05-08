# `org.panl.debug`

- **Namespace**: `org.panl.debug`
- **Status**: official
- **Version**: `schemaVersion = "1.0"`
- **Predecessor**: `org.kpow.debug` (read-side alias)

## Purpose

The single shared diagnostics / log extension. Any PANL component —
the converter, validator, benchmark, panel detector, app
metadata-enrichment pipeline — appends structured entries here so
tooling can audit how the file got the way it is.

Apps that don't care about diagnostics ignore the whole extension;
apps that do log into it without each one inventing a new extension
namespace per category.

This is the home for what historically lived in the `PROV`
(provenance) chunk and in legacy detector dumps under
`org.kpow.panel-detections`.

## Required / optional

Optional. Readers MUST be able to display a comic with the debug
extension entirely missing.

## Fields

```json
{
  "schemaVersion": "1.0",
  "entries": [
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
        "appVersion": "1.0.0"
      }
    },
    {
      "id": "dbg-1777346178400",
      "createdAtMs": 1777346178400,
      "source": "com.panel-flip",
      "category": "panel-detection",
      "level": "info",
      "summary": "Auto panel detection completed for 23 pages.",
      "data": {"detector": "opencv_v1", "pagesAnalyzed": 23}
    }
  ]
}
```

| Top-level field | Type                | Notes                                                  |
|-----------------|---------------------|--------------------------------------------------------|
| `schemaVersion` | string              | Required. `"1.0"`.                                     |
| `entries`       | array of entry      | Required. May be empty.                                |

### Entry fields

| Field         | Required | Type     | Notes                                                |
|---------------|----------|----------|------------------------------------------------------|
| `id`          | yes      | string   | Stable identifier; usually `dbg-<createdAtMs>`.       |
| `createdAtMs` | yes      | integer  | Epoch milliseconds.                                   |
| `source`      | yes      | string   | Logical producer — e.g. `org.panl.converter`, `org.panl.validator`, `org.panl.benchmark`, `com.panel-flip`. |
| `category`    | yes      | string   | What the entry's about (see below).                   |
| `level`       | yes      | enum     | `trace` / `debug` / `info` / `warning` / `error`.     |
| `summary`     | yes      | string   | One-line human-readable description.                  |
| `data`        | no       | object   | Free-form JSON payload. Use this for raw detector output, candidate codec stats, full provenance trees, etc. |

### Common categories

`conversion`, `metadata-import`, `metadata-enrichment`,
`panel-detection`, `panel-edit`, `benchmark`, `validation`,
`codec`, `streamability`, `extension`, `app`, `compat-migration`.

Apps may use any string. The validator accepts unknown categories
silently.

## Verbosity modes

The reference converter exposes `--debug none / summary / full`:

| Mode      | Behavior                                                       |
|-----------|----------------------------------------------------------------|
| `none`    | No debug extension written. Pre-existing debug entries are still preserved if the file is later edited. |
| `summary` | (Default) Compact import provenance + panel-detection summary entries. |
| `full`    | Verbose tracing (codec ladder details, per-tile quality gate results, etc.). |

Use `panl debug FILE [--level WARN] [--category panel-detection]
[--json]` to inspect entries.

## Rules

1. Debug entries are **optional**. Readers MUST be able to display a
   comic with this extension entirely missing.
2. Debug data MUST NOT be the only copy of user-facing data. Load-
   bearing fields (metadata, panels, cover, reading direction)
   belong in their domain extensions, not here.
3. Tools SHOULD preserve the debug extension on save unless the user
   explicitly chooses to strip it.
4. Unknown debug categories are accepted without warning.
5. Apps SHOULD log into the shared extension rather than inventing
   `com.example.app-debug` — use the `source` field instead.

## Validation rules

| Check                                                  | Severity |
|--------------------------------------------------------|----------|
| Payload not an object                                  | error    |
| `schemaVersion` missing                                | warning  |
| `entries` not an array                                 | error    |
| Entry missing any required field                       | error    |
| `level` not in the allowed set                         | warning  |
| `createdAtMs` not a non-negative integer               | error    |
| Duplicate `id` within `entries`                        | warning  |
| Total payload exceeds 10% of EDIT capacity             | warning (informational) |

## Preservation rules

- Writers MUST preserve unknown entries round-trip — including
  entries with unknown `source`, `category`, or extra `data` fields.
- Writers MAY trim the debug extension on user request (`--strip-debug`).
- Writers MUST NOT silently drop entries on save in normal operation.

## Compatibility notes

- Legacy namespace: `org.kpow.debug`.
- The legacy `PROV` chunk and `org.kpow.panel-detections` extension
  fold into this extension during migration. See
  [../../spec/COMPATIBILITY.md](../../spec/COMPATIBILITY.md).
