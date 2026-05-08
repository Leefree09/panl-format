# PANL extensions

PANL Core is small. Almost everything beyond *"this is a comic with
N pages in this order"* lives in **extensions** — namespaced
sub-objects inside the EDIT overlay's `extensions` map. This
directory catalogues them.

## Why extensions

PANL is not one perfect fixed comic schema. It's an open, modular
container that apps shape around their needs. The image package
(header, manifest, page index, page chunks) is immutable; the EDIT
overlay holds mutable comic intelligence; extensions inside that
overlay are how features are added without changing the core spec.

## Rules at a glance

- **EDIT is the extension container.** Every official, draft, and
  private extension lives under `EDIT.extensions[<namespace>]`.
- **Official PANL extensions use `org.panl.*`.** Their stable
  catalogue is in [official/](official/).
- **Apps may define private extensions** under reverse-domain
  namespaces (`com.panel-flip`, `dev.example.reader`,
  `io.acme.viewer`). See [private/README.md](private/README.md).
- **Unknown extensions MAY be ignored by readers.** A reader doesn't
  have to understand every namespace it sees — it can render the
  comic from core + the extensions it knows.
- **Unknown extensions MUST be preserved by writers.** Apps that
  round-trip through PANL MUST NOT delete extensions they don't
  understand. This is the most important rule in the extension
  model.
- **Official extensions are optional unless a compatibility profile
  requires them.** A reader without `org.panl.bookmarks` MUST still
  open and render a file that includes it.
- **Extension names are stable.** Once an `org.panl.*` extension is
  promoted from [draft/](draft/) to [official/](official/), its
  namespace and required fields don't change in a backward-
  incompatible way without a `schemaVersion` bump and a transition
  plan documented in [../spec/COMPATIBILITY.md](../spec/COMPATIBILITY.md).

The full container-level rules are in
[../spec/EDIT.md](../spec/EDIT.md).

## Layout

```
extensions/
  README.md                ← this file
  official/                ← stable org.panl.* extensions
    metadata.md
    presentation.md
    preferences.md
    panels.md
    bookmarks.md
    page-roles.md
    debug.md
  draft/                   ← reserved namespaces, not yet stable
    annotations.md
    reading-progress.md
    store-links.md
    translations.md
    accessibility.md
  private/                 ← guidance for app-private namespaces
    README.md
```

JSON Schemas for the official extensions live in
[../schemas/extensions/](../schemas/extensions/).

## Extension status taxonomy

Each extension declares one of:

| Status        | Meaning                                                                   |
|---------------|---------------------------------------------------------------------------|
| `official`    | Stable. Readers and writers expected to follow the spec exactly.          |
| `draft`       | Namespace reserved; spec not yet stable. Don't ship under the official name. Use a private namespace until the draft is promoted. |
| `deprecated`  | Spec retained for read-side compatibility; new files SHOULD NOT use it.   |
| `private`     | App-defined extension under a reverse-domain namespace. Out of scope here. |

## What every official extension spec contains

The pages under [official/](official/) follow a consistent template.
Each page documents:

- **Namespace** — the canonical key (e.g. `org.panl.metadata`).
- **Status** — `official` for everything in this folder.
- **Version** — current `schemaVersion`.
- **Purpose** — a one-paragraph description of what the extension is
  for and what it isn't.
- **Required / optional** — whether the extension MUST be present
  for a file to be considered conformant under any compatibility
  profile (most are optional).
- **Fields** — every field the extension defines, with type and
  semantics.
- **Examples** — at least one minimal and one fuller example.
- **Validation rules** — error / warning table specific to the
  extension.
- **Preservation rules** — what writers MUST preserve, what they MAY
  drop.
- **Compatibility notes** — legacy field names, `org.kpow.*`
  predecessors, migration behavior.

Per-extension JSON Schemas live in
[../schemas/extensions/](../schemas/extensions/).

## Adding a new official extension

1. Open a draft in [draft/](draft/) under the proposed name.
2. Reserve the namespace — apps that want to experiment use a
   private namespace until the draft promotes.
3. Once stable, move the doc to [official/](official/), add a JSON
   Schema under [../schemas/extensions/](../schemas/extensions/),
   and bump the spec date in [../CHANGELOG.md](../CHANGELOG.md).
4. Add a row to the effective-value rules in
   [../spec/EDIT.md](../spec/EDIT.md) if the extension contributes
   to one.

## Adding a private / app extension

You don't need to ask anyone. Pick a reverse-domain namespace you
control and put your data there. See
[private/README.md](private/README.md) for the full guidance.
