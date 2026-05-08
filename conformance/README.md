# PANL conformance suite

This directory holds the reference conformance vectors a PANL
implementation should pass. The tests are language-agnostic: each
test is a small input artefact (a `.panl` file or a manifest of
operations) and an expected outcome (a JSON document describing what
a conformant reader/validator MUST produce).

## Layout

```
conformance/
  README.md                ← this file
  tests/                   ← test inputs (.panl files + per-test READMEs)
  expected/                ← expected outputs, one .json per test
```

Each test has a stable id (e.g. `core-classic-minimal`,
`edit-ext-roundtrip-unknown`, `compat-kpow-legacy-flat`). Inputs
live at `tests/<id>/`; expected outputs at `expected/<id>.json`.

## Categories of test (planned)

The conformance suite is being built out alongside the v1.0 spec
freeze. The categories below are what's planned.

### Core container

- `core-classic-minimal` — smallest valid classic file. One PAGE,
  manifest, page index. Validator reports zero errors / warnings.
- `core-streamable-minimal` — smallest valid streamable file.
  Validator reports zero errors / warnings; BOOT chunk parses.
- `core-recovered-from-footer` — header `directory_offset` zeroed;
  reader recovers via footer; reports
  `recovered_from_footer = true`.
- `core-bad-header-bad-footer` — both pointers corrupt; reader
  rejects the file.
- `core-deep-crc-fail` — chunk CRC corrupted; deep validation
  rejects.

### Page index

- `pind-out-of-range-cover` — `coverPage` past `pageCount`.
- `pind-tile-overlap` — overlapping tile rectangles inside a page.
- `pind-mismatched-pagecount` — manifest disagrees with page index.

### EDIT / extensions

- `edit-ext-shape` — full extension shape; all seven official extensions
  populated; validator reports zero errors.
- `edit-ext-roundtrip-unknown` — file with a private extension
  `com.example.fizz`; reader/writer round-trips it byte-equivalent.
- `edit-ext-not-namespaced` — extension key `myExtension`; warning.
- `edit-ext-not-an-object` — extension payload is a string; error.
- `edit-overlay-overflow` — write that doesn't fit capacity; writer
  refuses.

### Profiles (informational, not strictly conformance)

- `profile-recommended-roundtrip` — sample produced by the
  Recommended profile; readers MUST decode every page.
- `profile-original-bytes-preserved` — Original profile; extracted
  page bytes are byte-identical to the source CBZ entries.

### Compatibility

- `compat-kpow-legacy-flat` — file with the pre-extension flat shape
  AND `manifest.format = "KPOW"`. Reader presents the same
  effective metadata as a current PANL file.
- `compat-kpow-namespaces` — file with `org.kpow.metadata` instead
  of `org.panl.metadata`. Reader merges; writer migrates on save and
  logs to `org.panl.debug` (`compat-migration`).

### Optional features

- `feat-dedup-roundtrip` — DDUP map; reader returns shared chunk
  bytes correctly.
- `feat-delt-roundtrip` — DELT chunk reconstruction matches the
  reference TILE pixels.
- `feat-palt-roundtrip` — PALT palette tile decodes to the recorded
  pixel hash.
- `feat-required-features-unknown` — `requiredFeatures` includes a
  name the reader doesn't understand; reader refuses the file.

## Expected-output format

```json
{
  "test": "core-classic-minimal",
  "validator": {
    "errors": [],
    "warnings": []
  },
  "reader": {
    "pageCount": 1,
    "coverPage": 0,
    "format": "PANL",
    "extensionsPresent": ["org.panl.metadata"]
  }
}
```

The exact shape will be finalised alongside the first batch of
fixtures. Reference implementations consume `expected/<id>.json`,
run their reader/validator on `tests/<id>/`, and compare.

## Status

No fixtures have been committed yet. This directory is a placeholder
for the v1.0 spec freeze.
