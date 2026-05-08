# PANL validation rules

Defines what a conformant PANL validator checks. Some checks are
errors (the file is not valid); some are warnings (the file may load
but is suspect). Per-extension validation is in each extension's
spec under [../extensions/official/](../extensions/official/).

A validator MAY support extra checks, but MUST flag everything in
the error column below as an error.

## Levels

- **error**: The file violates the spec. A conformant reader MAY
  refuse to render. `panl validate` exits non-zero.
- **warning**: Suspicious but tolerable. `panl validate` reports it
  and exits zero unless run with `--strict`.

## Container errors

| Check                                                            | Severity |
|------------------------------------------------------------------|----------|
| Header magic mismatch                                            | error    |
| Header CRC mismatch                                              | error    |
| Footer magic mismatch                                            | error    |
| Footer CRC mismatch                                              | error    |
| Header `directory_offset == 0` AND footer also invalid           | error    |
| Directory CRC mismatch                                           | error    |
| Duplicate chunk id in directory                                  | error    |
| Chunk overlaps header / directory / footer / file end            | error    |
| Two chunks overlap each other                                    | error    |
| Chunk `encoding != 0` (raw)                                      | error    |
| Per-chunk CRC mismatch (deep mode)                               | error    |
| Reader recovered the directory pointer from the footer           | warning  |

## Manifest errors

| Check                                                            | Severity |
|------------------------------------------------------------------|----------|
| Missing `manifest` chunk                                         | error    |
| Missing `page_index` chunk                                       | error    |
| Manifest `format` not in {`PANL`, `KPOW`}                        | error    |
| Manifest `version.major != 1`                                    | error    |
| Manifest `pageCount` mismatches `page_index` length              | error    |
| Manifest `coverPage` out of range                                | error    |
| Manifest `defaultReadingDirection` not in {`ltr`, `rtl`, `ttb`, `webtoon`, `unknown`} | warning |
| Manifest declares a `requiredFeatures` value the validator doesn't recognise | error |
| Manifest `features.metadata=true` but no `metadata` source       | warning  |
| Manifest `features.thumbnails=true` but no `thumb_index` chunk   | warning  |
| Manifest `features.panels=true` but no panels source             | warning  |

The "metadata source" and "panels source" warnings consider both the
EDIT extension (`org.panl.metadata`, `org.panl.panels`) and the
legacy chunks (`META`, `PANE`).

## Page index errors

| Check                                                            | Severity |
|------------------------------------------------------------------|----------|
| Page entry missing `chunkId`                                     | error    |
| Page `chunkId` does not resolve                                  | error    |
| Page chunk has type other than `PAGE` / `TILE`                   | error    |
| Page `width` / `height` present but ≤ 0                          | error    |
| Page `thumbnailChunkId` references a missing chunk               | error    |
| Tiled page has overlapping or out-of-bounds tile rectangles      | error    |
| Page entry missing optional fields (`width`, `height`, `codec`)  | warning  |
| Page `role` not in the allowed set                               | warning  |
| Page or thumbnail `codec` not in the allowed set                 | warning  |

Allowed `role`: `cover`, `story`, `ad`, `credits`, `backmatter`,
`watermark`, `extra`, `unknown`. Allowed `codec`: `jpeg`, `png`,
`webp`, `avif`, `jxl`, `unknown`, `palette`.

## Thumbnail index

| Check                                                            | Severity |
|------------------------------------------------------------------|----------|
| Thumbnail entry references missing chunk                         | error    |
| Thumbnail entry references a page out of range                   | error    |
| Thumbnail chunk has type other than `THUM`                       | warning  |
| Two thumbnails point at the same page                            | warning  |

## EDIT overlay

The container-level rules are in [EDIT.md](EDIT.md#validator-rules-for-edit).
Per-extension validators run after the container rules pass; each
official extension defines its own error / warning table under
[../extensions/official/](../extensions/official/).

Extension-related warnings:

| Check                                                            | Severity |
|------------------------------------------------------------------|----------|
| Extension key not namespaced (e.g. `myExtension`)                | warning  |
| Extension payload missing `schemaVersion`                        | warning  |
| Extension payload not an object                                  | error    |
| Two extensions claim the same conceptual data without an authority order resolution | warning |
| Legacy `org.kpow.*` extension key present                        | warning (informational; migrate via `panl migrate-edit`) |

## Custom (CUST) chunks

| Check                                                            | Severity |
|------------------------------------------------------------------|----------|
| CUST chunk present (consumer-unknown by design)                  | warning  |
| CUST chunk id missing namespace separator                        | warning  |

## Strict mode

`panl validate --strict` upgrades selected warnings to errors:

- Unknown chunk type code.
- CUST chunks present.
- Legacy `org.kpow.*` extension keys present.
- Missing `schemaVersion` on an extension payload.

## Deep mode

`panl validate --deep` re-reads every chunk and verifies its CRC-32
— the most expensive but deepest integrity check. Use this for
fsck-style flows.

## Optional-feature validation

Optional features (DDUP, DELT, PALT) have their own validator
tables in the reference implementation. They are part of v1 but not
part of the minimum reader contract. A reader that doesn't
understand them MUST refuse the file when the unknown feature
appears in `manifest.requiredFeatures`; otherwise it MAY ignore the
feature's chunks (which it sees as unknown chunk types it tolerates).
