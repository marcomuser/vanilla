# 0003 No build step

## Context

What the server sends is what the developer wrote. Files still have to load
fast and stay cached, and third-party code has to arrive the same way.

## Decision

**The server serves app code and third-party code as written, from its own
origin, at URLs that change whenever the content does.**

- App files are served at a URL that contains a hash of their content, such
  as `/static/3f9c2a/components/card/card.css`.
- Third-party packages are copied into the project at a pinned version and
  served under a path with that version, such as
  `/vendor/webawesome@3.13.0/…`.
- Versioned files are cached for a year as immutable. Documents are
  revalidated on every request.
- Modules import each other by name through an import map that the server
  writes into each page. Every import resolves to a versioned URL.
- Everything is served over HTTP/2 or later.

## Rationale

- **No toolchain.** The code you read is the code that runs.
- **Content in the URL makes caching safe.** A URL never changes what it
  serves, so it can be cached forever. A change gets a new URL. In the
  prototype, every stylesheet came from the cache on reload.
- **Per file, not per release.** A change invalidates only the files it
  touches.
- **Relative imports would bypass the hashes.** A module at a hashed URL that
  imports `./x.js` requests `x.js` under its own hash, not under the hash of
  `x.js`. The import map points each name at the right URL.
- **Own origin, not a public CDN.** Browsers partition their HTTP cache by
  site, so a public CDN gives no cache shared across sites. A CDN on another
  origin also needs its own connection, so its files do not share the page's
  HTTP/2 connection.

## Consequences

- The server hashes files at runtime and keeps the hashes until a file
  changes.
- The import map comes before every module script in `<head>`, from the same
  helper that links the page's stylesheets and definitions.
- Upgrading a package means copying the new version into the project. Its
  version in the path changes all its URLs.
- Third-party packages must ship ES modules that run in the browser as
  published, or the import map maps their bare imports.
- HTTP/2 is part of the server contract.

## Alternatives

- **A bundler:** a toolchain to maintain, and concatenation is no faster
  over HTTP/2 ([0006](0006-scoped-css-per-component.md)).
- **A public CDN:** no shared cache, an extra connection, and a third party
  on the critical path.
- **Version in the query string:** relative imports drop the query, so
  imported modules load unversioned.
- **One version per release in the path:** relative imports work, but every
  deploy invalidates every file.
- **Unversioned URLs with revalidation:** a round trip per file on every
  page.
