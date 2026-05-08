# `org.panl.store-links` *(draft)*

- **Namespace**: `org.panl.store-links` *(reserved)*
- **Status**: draft — not yet specified. Don't ship under the
  official name. Use a private namespace
  (e.g. `com.example.store-links`) until this draft is promoted.

## Intent

Vendor / purchase links: where to buy this issue, where the source
file came from, links to the publisher's storefront, affiliate URLs.
Useful for libraries that want to surface "buy this issue" / "more
from this publisher" affordances.

## Open design questions

- URL safety. Files are shared; embedded URLs become a permanent
  part of the artifact. Should the extension carry a hash or
  signature so a reader can detect tampering?
- Affiliate codes vs canonical store URLs. Probably both, kept
  separate so a reader can strip the affiliate code if the user
  prefers.
- Localisation: a single comic might have different storefronts per
  region. List of `(region, url)` tuples?
- Provenance: the URL of the *source file* (where this digital copy
  was bought) vs the URL of the *product page*. Different audiences
  want different things.

## Interim guidance

Apps that need store links today SHOULD use a reverse-domain
namespace (`com.<app>.store-links`) and migrate once promoted.
