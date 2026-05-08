# PANL streamable layout

Defines the bootstrap (BOOT) chunk used by the streamable file
layout. Streamable PANL files front-load core metadata so a network
or library reader can fetch startup state with one or two range
reads.

The on-disk envelope and the streamable section diagram are in
[CONTAINER.md](CONTAINER.md). This document covers the BOOT chunk
itself.

## When streamable is in effect

A file is streamable when bit 0 of the header `flags` field
(`FLAG_STREAMABLE`, `0x1`) is set. Such a file MUST contain a single
`BOOT` chunk located immediately after the header (at byte offset
`64`).

A reader that finds `FLAG_STREAMABLE` clear MUST NOT look for a BOOT
chunk: classic files are free to omit it entirely.

## BOOT payload binary header

The BOOT chunk is a fixed-size reservation written by the writer up
front and patched in place at finalize, after the directory is
known. The actual JSON payload may be smaller than the reservation;
the trailing bytes are zero-padded.

```
4 bytes   "KBOT"          payload magic
2 bytes   bootstrap_format_version (uint16, currently 1)
4 bytes   reserved                  (uint32, 0)
4 bytes   json_length               (uint32) — bytes of JSON that follow
N bytes   json_bytes                (compact UTF-8 JSON)
4 bytes   bootstrap_crc32           (CRC-32 over all preceding bytes)
```

`json_length` is authoritative; readers MUST trust it over the
chunk's outer `length` (which is the on-disk reservation).

The BOOT chunk's directory entry uses the `SKIP_CRC_SENTINEL` of `0`;
the chunk verifies itself via `bootstrap_crc32`. See
[CONTAINER.md](CONTAINER.md#crcs).

## BOOT JSON document (v1)

```json
{
  "bootstrapVersion": 1,
  "format": "PANL",
  "version": {"major": 1, "minor": 0},
  "layout": "streamable",
  "pageCount": 44,
  "coverPage": 0,
  "thumbnailMode": "cover",
  "requiredCodecs": ["avif", "webp"],
  "storageModes": ["tiled"],
  "chunks": {
    "manifest":       {"type": "MANI", "id": "manifest",     "offset": ..., "length": ...},
    "pageIndex":      {"type": "PIND", "id": "page_index",   "offset": ..., "length": ...},
    "thumbnailIndex": {"type": "TIND", "id": "thumb_index",  "offset": ..., "length": ...},
    "coverThumbnail": {"type": "THUM", "id": "thumb_cover",  "offset": ..., "length": ...},
    "editOverlay":    {"type": "EDIT", "id": "edit_overlay",
                       "offset": ..., "length": ..., "usedLength": ...},
    "directory":      {"offset": ..., "length": ...}
  },
  "pageChunkOffsets": [
    {"id": "page_0000", "type": "PAGE", "offset": ..., "length": ..., "crc32": ...}
  ]
}
```

Optional fields (any may be absent):

- `pageCount`, `coverPage`, `thumbnailMode`, `requiredCodecs`,
  `storageModes` — informational mirrors of the manifest, useful for
  library scans.
- `pageChunkOffsets[]` — per-PAGE/per-TILE pointer table so a range
  reader can fetch any page or tile by chunk id without paging in the
  directory. Entries SHOULD include the chunk's CRC-32 so a reader
  can verify in isolation.
- `chunks.editOverlay` — for streamable + EDIT files, the BOOT
  pointer carries an extra `usedLength` field. It is informational —
  it lets a range reader fetch only the live bytes
  (`offset` .. `offset + EDIT_HEADER_SIZE + usedLength`) instead of
  the whole reservation. The chunk's internal header is still
  authoritative; `usedLength` may go stale across in-place edits and
  that's fine.
- Pointers to optional features (`dedupMap`, etc.) MAY also appear
  here for tools that need them.

Required:

- `chunks.manifest` and `chunks.pageIndex` MUST be present.
- `chunks.directory` MUST agree with the header's `directory_offset`
  and `directory_length`. Disagreement is a hard error: the reader
  rejects the bootstrap and falls back to classic directory parsing.

## Read-time behavior

A reader that finds `FLAG_STREAMABLE` set:

1. Reads the header (64 bytes).
2. Reads the BOOT chunk header at offset 64 (14 bytes) to learn
   `json_length`; reads `json_length + 4` more bytes; verifies the
   bootstrap CRC.
3. Uses BOOT pointers to read manifest / page index / cover
   thumbnail directly, in any order.
4. Uses `pageChunkOffsets[]` (or, when absent, the chunkIds in the
   page index) to read page/tile bytes by offset.
5. Loads the full directory only when needed for validation, chunk
   listing, or CRC verification.

A reader that finds `FLAG_STREAMABLE` set but cannot parse the BOOT
chunk (bad magic, bad CRC, directory pointer mismatch) MUST fall
back to the classic directory-at-end path, expose the failure
through reader API (e.g. `bootstrap_error`), and surface an error
from `validate()`.

## Backward compatibility

- A streamable file remains a valid PANL v1 file: a classic-only
  reader can ignore the streamable bit and the BOOT chunk (which it
  sees as an unknown chunk type, which the spec already requires
  readers to tolerate).
- A classic file remains a valid PANL v1 file under the streamable
  rules: `FLAG_STREAMABLE` is clear and no BOOT chunk is expected.
- Existing classic files do not need to be re-encoded. A
  `panl restream` operation produces a streamable copy in a single
  forward pass without modifying any user chunk bytes.
