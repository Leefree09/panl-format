# PANL — open comic container format

Status: v1 reference. Approaching the v1.0 spec freeze.

PANL is a page-oriented comic container that replaces the
CBZ/CBR/PDF + sidecar metadata workflow with a single structured
file: explicit page order, embedded metadata, embedded panel layouts,
embedded thumbnails, and chunked storage with fast random access via
a directory.

PANL was previously known as **KPOW** during early development.
The on-disk shape is the same. See [COMPATIBILITY.md](COMPATIBILITY.md)
for migration rules.

## How this spec is organised

PANL Core is intentionally small. The specification is split into
focused documents:

| Document                              | Defines                                           |
|---------------------------------------|---------------------------------------------------|
| [CONTAINER.md](CONTAINER.md)          | Header, footer, chunk directory, CRCs.            |
| [CHUNKS.md](CHUNKS.md)                | Chunk types, ids, encoding, per-chunk schemas.    |
| [STREAMABLE.md](STREAMABLE.md)        | `FLAG_STREAMABLE` and the BOOT chunk.             |
| [EDIT.md](EDIT.md)                    | The editable overlay container and extension rules. |
| [PROFILES.md](PROFILES.md)            | User-facing conversion profiles.                  |
| [VALIDATION.md](VALIDATION.md)        | Validator rules and severity levels.              |
| [COMPATIBILITY.md](COMPATIBILITY.md)  | Versioning and KPOW → PANL migration.             |

Per-extension specs live under [../extensions/](../extensions/).
JSON Schemas live under [../schemas/](../schemas/).

## What's in core

A PANL file always contains:

- A 64-byte **header** with format magic, version, flags, and a
  pointer to the chunk directory ([CONTAINER.md](CONTAINER.md)).
- A 32-byte **footer** that duplicates the directory pointer for
  recovery ([CONTAINER.md](CONTAINER.md)).
- A **chunk directory** indexing every chunk by id, type, offset,
  length, and CRC-32 ([CONTAINER.md](CONTAINER.md)).
- A **manifest** chunk (`MANI`) declaring page count, cover, default
  reading direction, and feature flags ([CHUNKS.md](CHUNKS.md)).
- A **page index** chunk (`PIND`) with the authoritative reading
  order ([CHUNKS.md](CHUNKS.md)).
- One or more **page** (or **tile**) chunks holding image bytes.
- An optional **EDIT overlay** holding mutable comic intelligence
  via namespaced extensions ([EDIT.md](EDIT.md)).
- An optional **BOOT chunk** for streamable layout
  ([STREAMABLE.md](STREAMABLE.md)).

What's *not* in core: bibliographic metadata schema, panel layouts,
page roles, reading preferences, bookmarks, reading progress,
annotations, store links — all of these live in **extensions**
under [EDIT.md](EDIT.md). Apps are expected to compose the format
out of core + the official extensions they need + their own private
extensions.

## On-disk basics

- All multi-byte integers are **little-endian**.
- All strings are **UTF-8**.
- The image package (header, manifest, page/tile chunks, directory,
  footer) is **immutable** once written. Edits land in the EDIT
  overlay reservation, never in image bytes.
- Readers MUST tolerate unknown chunk types and unknown extension
  namespaces. Writers MUST preserve them on round-trip.

## Versioning

The format header carries `version_major` and `version_minor`. PANL
v1 corresponds to KPOW v1. A reader that rejects `version_major != 1`
is correct for this spec. See [COMPATIBILITY.md](COMPATIBILITY.md)
for the long-term policy.
