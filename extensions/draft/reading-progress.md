# `org.panl.reading-progress` *(draft)*

- **Namespace**: `org.panl.reading-progress` *(reserved)*
- **Status**: draft — not yet specified. Don't ship under the
  official name. Use a private namespace
  (e.g. `com.example.reading-progress`) until this draft is
  promoted.

## Intent

Per-comic reading progress — last page read, last panel read,
percentage complete, "finished" flag. Used by readers to resume
where the user left off.

## Open design questions

- Single-user vs multi-user state. Many comic readers are personal,
  but a shared-library setup (e.g. a household tablet) needs a way
  to keep different users' progress separate. Likely answer: this
  extension holds *one* progress record; per-user state lives in the
  app's user database, not in the file.
- Granularity: page-level vs panel-level vs scroll-position. Webtoon
  and panel-mode readers want sub-page positions; classic readers
  only need page index.
- Concurrency: if the file is sync'd across devices, how do
  conflicting writes resolve? Probably last-writer-wins keyed by
  `updatedAtMs`.
- Should "read once" history be retained, or only the latest
  position? Privacy + size tradeoff.

## Interim guidance

Apps that need reading progress today SHOULD use a reverse-domain
namespace (`com.<app>.reading-progress`) and migrate once promoted.
