# `org.panl.preferences`

- **Namespace**: `org.panl.preferences`
- **Status**: official
- **Version**: `schemaVersion = "1.0"`
- **Predecessor**: `org.kpow.preferences` (read-side alias)

## Purpose

Per-comic reading defaults: reading direction, start page, spread
mode, page-fit mode, panel mode default. These are hints to the
reader app — what *this comic* prefers when it's opened. They are
not user-account-level preferences; those live in the app, not the
file.

## Required / optional

Optional. Readers fall back through the effective-value rules in
[../../spec/EDIT.md](../../spec/EDIT.md).

## Fields

```json
{
  "schemaVersion": "1.0",
  "readingDirection": "ltr",
  "startPage": 0,
  "spreadMode": "auto",
  "pageFitMode": "best",
  "panelModeDefault": false,
  "preferredPanelExtension": "org.panl.panels",
  "custom": {}
}
```

| Field                       | Type                | Notes                                                  |
|-----------------------------|---------------------|--------------------------------------------------------|
| `schemaVersion`             | string              | Required. `"1.0"`.                                     |
| `readingDirection`          | enum string         | Optional. One of `ltr`, `rtl`, `ttb`, `webtoon`, `unknown`. Overrides `manifest.defaultReadingDirection`. |
| `startPage`                 | integer             | Optional. Page index to open on. MUST be in range `[0, pageCount)`. |
| `spreadMode`                | enum string         | Optional. One of `single`, `double`, `auto`.           |
| `pageFitMode`               | enum string         | Optional. One of `best`, `width`, `height`, `original`. |
| `panelModeDefault`          | boolean             | Optional. If `true`, readers SHOULD open in panel mode by default. |
| `preferredPanelExtension`   | string (namespace)  | Optional. Namespace of the panel extension a reader should prefer when more than one is present (e.g. `org.panl.panels` vs a private detector). |
| `custom`                    | object              | Optional. App-specific extras.                         |

## Validation rules

| Check                                                  | Severity |
|--------------------------------------------------------|----------|
| Payload not an object                                  | error    |
| `schemaVersion` missing                                | warning  |
| `readingDirection` not in allowed set                  | warning  |
| `startPage` not an integer / out of range              | error    |
| `spreadMode` not in allowed set                        | warning  |
| `pageFitMode` not in allowed set                       | warning  |
| `panelModeDefault` not boolean                         | error    |
| `preferredPanelExtension` references an extension that isn't present | warning |

## Preservation rules

- Writers MUST preserve unknown top-level keys round-trip.
- Writers MUST preserve `custom` round-trip.

## Compatibility notes

- Legacy namespace: `org.kpow.preferences`.
