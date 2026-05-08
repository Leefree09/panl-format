# Private / app-specific extensions

PANL apps can store their own data inside a `.panl` file without
asking permission and without coordinating with the spec. The rules
are simple, and they exist so that PANL files round-trip cleanly
between apps that don't know about each other.

## Pick a namespace you control

Use a reverse-domain namespace under a domain or org you actually
own:

- `com.panel-flip` — the Panel Flip reader app
- `dev.example.reader` — a reader at `example.dev`
- `io.acme.viewer` — a viewer at `acme.io`

Don't use:

- `org.panl.*` — reserved for official extensions.
- Generic, non-namespaced keys like `myExtension` (validators warn).
- Other vendors' namespaces.

If you don't have a domain, use a clearly-app-scoped namespace
(`app.<unique-name>.<feature>`) and accept that it may collide
later.

## Extension shape

Same rules as any other extension. Your payload sits at
`EDIT.extensions["<your-namespace>"]` and SHOULD include
`schemaVersion`:

```json
{
  "schemaVersion": "1.0",
  "...": "..."
}
```

You're free to put anything inside the payload that's a valid JSON
object. Validators won't flag the contents — they only check that
the key is namespaced and the payload is an object.

## Round-trip and preservation

This is the rule that makes the ecosystem work:

> **Apps MUST preserve unknown extensions on save.**

If your app reads a `.panl` file written by a different app and
saves it back out, the other app's namespaced extension MUST still
be there byte-equivalent (or as close as JSON re-serialization
allows). Don't strip extensions just because you don't recognise
them. Don't rewrite their internal shape.

The reference reader/writer libraries handle this automatically.
If you write your own writer, treat unknown extensions as opaque.

## Relating to official extensions

If your data overlaps an official extension, prefer the official
one:

- Bibliographic metadata → put it in `org.panl.metadata`, with
  app-specific extras under `org.panl.metadata.custom["<your-ns>"]`.
- Panel layouts → contribute to `org.panl.panels` if your panels are
  intended for general reading; use a private extension only when
  your data is app-internal.
- Diagnostics / logs → log into `org.panl.debug` with `source =
  "<your-namespace>"`. Don't invent `com.example.app-debug`.

Use a private extension when:

- The feature isn't covered by any official extension.
- The feature is in [draft/](../draft/) but you need to ship today.
- The data is genuinely app-private (license tokens, internal
  caches, app-specific UI state) and not something other apps
  should consume.

## When a draft promotes to official

If your private extension's domain later becomes an official
`org.panl.*` extension (e.g. `com.example.bookmarks` →
`org.panl.bookmarks`), the official spec's compatibility section
will document the migration path — usually "readers MUST accept
both keys; writers SHOULD migrate on save".

Until then, your namespace is yours; the spec won't take it.
