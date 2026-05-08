# PANL container

Defines the on-disk envelope: header, footer, chunk directory, CRCs,
and the two layouts (classic and streamable).

All multi-byte integers are **little-endian**. All strings are
**UTF-8**.

## File layout

PANL supports two layouts. Classic is the original layout; streamable
front-loads core metadata so a network or library reader can fetch
startup data with one or two range reads.

### Classic layout (default)

```
Offset                              Region
──────────────────────────────────  ─────────────────────────────────────
0  ..  64                           Header              (64 bytes, fixed)
64 ..  directory_offset             Chunk payloads      (variable)
directory_offset .. footer_offset   Chunk directory     (variable)
file_size - 32 .. file_size         Footer              (32 bytes, fixed)
```

### Streamable layout

```
Offset                              Region
──────────────────────────────────  ─────────────────────────────────────
0  ..  64                           Header              (64 bytes, fixed)
64 ..  64 + R                       Bootstrap chunk     (R = reservation, fixed)
64 + R .. front_end                 Front-loaded JSON / index / cover
                                    (manifest, metadata, provenance,
                                     page index, thumbnail index, panels,
                                     cover thumbnail)
front_end .. directory_offset       Page / tile / per-page thumbnail bulk
directory_offset .. footer_offset   Chunk directory     (variable)
file_size - 32 .. file_size         Footer              (32 bytes, fixed)
```

The streamable header has bit 0 of `flags` (`FLAG_STREAMABLE`, `0x1`)
set. See [STREAMABLE.md](STREAMABLE.md) for the BOOT chunk shape and
read-time behavior.

## Write order

The header is written first with a placeholder zero directory
pointer. Chunks are appended as they are written. At `finalize()`:

1. The directory is appended at the current position.
2. The footer is appended.
3. For streamable files, the BOOT slot is patched in place with the
   real chunk offsets and the directory pointer.
4. The header is back-patched in place with the real directory
   pointer. The `FLAG_STREAMABLE` bit is set for streamable files.

The footer exists so a reader can recover when the header has been
clobbered — for example, a writer that crashed between steps 2 and 3.
Both header and footer carry the same `directory_offset` /
`directory_length`, each with its own CRC-32.

## Header (64 bytes)

| Offset | Size | Field             | Notes                                          |
|-------:|-----:|-------------------|------------------------------------------------|
|      0 |    8 | `magic`           | `KPOW\r\n\x1A\n` (kept opaque for v1; see [COMPATIBILITY.md](COMPATIBILITY.md)) |
|      8 |    2 | `version_major`   | uint16 — `1` for this spec                     |
|     10 |    2 | `version_minor`   | uint16 — `0` for this spec                     |
|     12 |    4 | `flags`           | uint32 — bit 0 = `FLAG_STREAMABLE`; other bits reserved, `0` |
|     16 |    8 | `directory_offset`| uint64 — byte offset of directory; `0` if not finalized |
|     24 |    8 | `directory_length`| uint64 — bytes; `0` if not finalized           |
|     32 |   28 | `reserved`        | zero-filled                                    |
|     60 |    4 | `header_crc32`    | CRC-32 of bytes `[0, 60)`                      |

The `\r\n`/`\x1A`/`\n` byte sequence in the magic catches text-mode
mangling on misbehaving toolchains.

A reader rejecting `version_major != 1` is correct for this spec.

If `directory_offset == 0` the file has not been finalized via the
header. Try the footer.

## Footer (32 bytes)

The footer always sits at `file_size - 32`. It duplicates the
directory pointer with an independent CRC.

| Offset | Size | Field              | Notes                                     |
|-------:|-----:|--------------------|-------------------------------------------|
|      0 |    8 | `footer_magic`     | `KPOWend\n` (opaque for v1)               |
|      8 |    8 | `directory_offset` | uint64 — same value the header carries    |
|     16 |    8 | `directory_length` | uint64 — same value the header carries    |
|     24 |    4 | `flags`            | uint32 — reserved, `0`                    |
|     28 |    4 | `footer_crc32`     | CRC-32 of bytes `[0, 28)`                 |

A v1 file MUST contain a valid footer. If a reader finds the footer
is invalid but the header is, it MUST reject the file (treat the file
as damaged); the footer is the redundancy guarantee, and silently
dropping the cross-check would defeat the point.

A reader MAY recover from a corrupt header if the footer is intact,
in which case it should expose this state (e.g.
`recovered_from_footer = True`).

## Chunk directory

Located at `directory_offset` and exactly `directory_length` bytes
long.

```
4 bytes   "KDIR"  magic
2 bytes   directory_version  (uint16, currently 1)
4 bytes   entry_count        (uint32)
[entries]
4 bytes   directory_crc32    (CRC-32 over all preceding directory bytes)
```

Each entry is variable-length:

| Size | Field        | Notes                                                |
|-----:|--------------|------------------------------------------------------|
|    4 | `type_code`  | 4 ASCII bytes (e.g. `MANI`, `PAGE`)                  |
|    2 | `id_length`  | uint16                                                |
|    N | `chunk_id`   | UTF-8 bytes, no null terminator                      |
|    8 | `offset`     | uint64 — byte offset of chunk payload                |
|    8 | `length`     | uint64 — byte length of chunk payload                |
|    1 | `encoding`   | uint8 — `0` = raw (uncompressed); other values reserved |
|    1 | `reserved`   | `0`                                                   |
|    2 | `flags`      | uint16 — reserved, `0`                                |
|    4 | `chunk_crc32`| CRC-32 of the chunk payload                          |

Chunk payloads have no per-chunk header on disk — the directory
entry is the sole descriptor. This keeps writes streamable: a single
forward pass plus one seek-back at finalize.

## CRCs

There are four independent CRCs in a finalized file:

1. **Header CRC** — protects the version, flags, and directory pointer.
2. **Footer CRC** — protects the redundant directory pointer at end of file.
3. **Directory CRC** — protects the list of chunk descriptors.
4. **Per-chunk CRCs** — protect each chunk payload (verified on read).

Two chunk types use a `SKIP_CRC_SENTINEL` of `0` in their directory
entry instead of a valid CRC:

- **EDIT** — the chunk carries its own internal `payload_crc32` so
  in-place edits don't have to keep the directory CRC in sync.
- **BOOT** — same reason. The reservation is patched at finalize and
  the BOOT payload has its own internal CRC.

Validators MUST treat the sentinel as valid for these chunk types
specifically; for any other type, a `0` CRC value is verified as a
real CRC.
