# PANL samples

This directory will hold small, illustrative `.panl` files that show
each profile and extension in action. Samples are for *people* —
they're meant to be opened, inspected, hex-dumped, and read by
implementers building a PANL reader from the spec.

The conformance suite under [../conformance/](../conformance/) is
the machine-checkable counterpart.

## Planned samples

- `minimal.panl` — smallest valid PANL file (one page, no metadata,
  classic layout).
- `streamable.panl` — same content as `minimal.panl` but streamable.
- `recommended.panl` — output of the `Recommended` profile on a
  small public-domain comic. Includes EDIT overlay with all six
  official extensions populated.
- `original.panl` — `Original` profile preserving source bytes
  verbatim.
- `with-private-extension.panl` — illustrates a private
  reverse-domain extension (`com.example.demo`) round-tripping
  through the standard tools.
- `legacy-kpow.panl` — file written by the pre-rename `kpow` writer.
  Demonstrates that PANL readers consume `.kpow` content unchanged.

## Naming and licensing

Sample files SHOULD use public-domain or freely-licensed source
images. The README in each sample subdirectory will document the
source, license, and the exact conversion command used to produce
it.

## Status

No samples have been committed yet. This directory is a placeholder
for the v1.0 spec freeze.
