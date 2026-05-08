# `org.panl.metadata`

- **Namespace**: `org.panl.metadata`
- **Status**: official
- **Version**: `schemaVersion = "1.0"`
- **Predecessor**: `org.kpow.metadata` (read-side alias)

## Purpose

Bibliographic / comic metadata — title, series, issue, publisher,
release date, credits, characters, identifiers. Equivalent to the
data that traditionally lived in a sidecar `ComicInfo.xml` or a
library-database row. This extension is the canonical home for
that data inside a PANL file.

It is **not** a place for app-specific bookkeeping (see
[debug.md](debug.md) and the `custom` sub-object below).

## Required / optional

Optional. A PANL file with no `org.panl.metadata` extension is valid
— readers fall back through the effective-metadata authority order
defined in [../../spec/EDIT.md](../../spec/EDIT.md#effective-metadata).

## Fields

```json
{
  "schemaVersion": "1.0",
  "title": "Hawkeye",
  "series": "Hawkeye",
  "issue": "1",
  "volume": "2012",
  "publisher": "Marvel",
  "imprint": null,
  "releaseDate": "2012-08-15",
  "description": "Clint Barton, the marksman known as Hawkeye…",
  "language": "en",
  "ageRating": "Teen",
  "characters": ["Clint Barton", "Kate Bishop"],
  "teams": [],
  "locations": ["Brooklyn"],
  "tags": ["street-level"],
  "genres": ["superhero"],
  "credits": [
    {"role": "writer", "names": ["Matt Fraction"]},
    {"role": "penciller", "names": ["David Aja"]},
    {"role": "colorist", "names": ["Matt Hollingsworth"]}
  ],
  "identifiers": {
    "comicVineVolumeId": 12345,
    "isbn": null,
    "diamondCode": null
  },
  "custom": {
    "com.panel-flip": {"matchScore": 0.93}
  }
}
```

| Field           | Type                       | Notes                                                     |
|-----------------|----------------------------|-----------------------------------------------------------|
| `schemaVersion` | string                     | Required. `"1.0"` for this version.                       |
| `title`         | string                     | Optional. Human-readable issue title.                     |
| `series`        | string                     | Optional. Series name.                                    |
| `issue`         | string                     | Optional. Issue number as string (allows `"1.5"`, `"½"`). |
| `volume`        | string                     | Optional. Volume designation.                             |
| `publisher`     | string                     | Optional.                                                 |
| `imprint`       | string \| null             | Optional.                                                 |
| `releaseDate`   | string (ISO 8601 date)     | Optional. `YYYY-MM-DD` preferred.                         |
| `description`   | string                     | Optional. Issue summary.                                  |
| `language`      | string (ISO 639-1)         | Optional. e.g. `"en"`, `"ja"`.                            |
| `ageRating`     | string                     | Optional. Free-form (`"Teen"`, `"Mature"`, `"Everyone"`). |
| `characters`    | array of string            | Optional.                                                 |
| `teams`         | array of string            | Optional.                                                 |
| `locations`     | array of string            | Optional.                                                 |
| `tags`          | array of string            | Optional.                                                 |
| `genres`        | array of string            | Optional.                                                 |
| `credits`       | array of credit object     | Optional. Each entry: `{role, names[]}`.                  |
| `identifiers`   | object                     | Optional. Free-form keyed identifiers (Comic Vine, ISBN, etc.). |
| `custom`        | object keyed by namespace  | Optional. Small app-specific extras related to metadata.  |

`credits[].role` is conventionally one of `writer`, `penciller`,
`inker`, `colorist`, `letterer`, `coverArtist`, `editor`, but readers
MUST accept arbitrary strings.

## Examples

### Minimal

```json
{
  "schemaVersion": "1.0",
  "series": "Hawkeye",
  "issue": "1"
}
```

### Imported from ComicInfo.xml

See [../../spec/COMPATIBILITY.md](../../spec/COMPATIBILITY.md) for
the import mapping. The full example above is what
`panl convert` produces from a CBZ that carries a complete
`ComicInfo.xml` plus a `.ComicMeta.json` from a Panel Flip library.

## Validation rules

| Check                                                  | Severity |
|--------------------------------------------------------|----------|
| Payload not an object                                  | error    |
| `schemaVersion` missing                                | warning  |
| `releaseDate` not parseable as `YYYY-MM-DD` (when present) | warning |
| `language` not a 2-letter ISO 639-1 code (when present) | warning |
| `credits[i]` missing `role` or `names`                 | error    |
| `credits[i].names` not an array of strings             | error    |
| `identifiers` value not string / number / boolean / null | warning |
| `custom` key not namespaced                            | warning  |

## Preservation rules

- Writers MUST preserve unknown top-level keys round-trip. (Forward-
  compatible field additions in v1.x will go here.)
- Writers MUST preserve the entire `custom` map round-trip, including
  unknown namespaces under it.
- Writers SHOULD NOT silently rewrite `identifiers` values they don't
  recognise.

## Compatibility notes

- Files written before the rename use the namespace
  `org.kpow.metadata` with the same shape. Readers MUST accept both;
  `org.panl.metadata` wins when both are present.
- Pre-extension flat files use a top-level `metadata` field on the
  EDIT JSON. The migration path is in
  [../../spec/COMPATIBILITY.md](../../spec/COMPATIBILITY.md).
- The legacy `META` chunk holds equivalent data when no EDIT
  extension is present. The effective-metadata rules in
  [../../spec/EDIT.md](../../spec/EDIT.md#effective-metadata) define
  the precedence.
