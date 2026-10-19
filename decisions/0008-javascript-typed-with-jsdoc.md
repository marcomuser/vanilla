# 0008 JavaScript typed with JSDoc

## Context

The server sends files as they were written ([0003](0003-no-build-step.md)).
Browsers run JavaScript, not TypeScript, so TypeScript source has to be
compiled before it can be served. As an app grows, its code still benefits
from type checking.

## Decision

**App code is JavaScript. Where the app wants types, it writes them in
JSDoc.**

- Modules are plain JavaScript files, served as written.
- `tsc` can check them in CI. It reads the JSDoc and emits nothing.
- `paths` in `tsconfig.json` mirror the import map.

## Rationale

- **JSDoc is a comment.** Annotated code is normal JavaScript, so the
  browser runs what the developer wrote.
- **`tsc` checks JSDoc like TypeScript.** It catches wrong argument types,
  missing null checks, imports that do not resolve and misspelled
  properties on design-system components, which ship their own types.
- **The checker resolves names as the browser does.** Imports go through the
  import map ([0003](0003-no-build-step.md)), which `tsc` does not read.
  `paths` give it the same names and versions.

## Consequences

- TypeScript is at most a development dependency. Serving and running the
  app need neither TypeScript nor Node.js.
- Inline scripts cannot be checked, because `tsc` does not read HTML. Code
  that needs checking lives in module files.
- The server writes the import map and `tsconfig.json` holds `paths`, so the
  same packages are listed in two places. Upgrading a package changes both.
- Types in JSDoc are more verbose than in TypeScript syntax.

## Alternatives

- **TypeScript with a compiler or bundler:** a build step, and the served
  code is not the code the developer wrote.
- **TypeScript stripped by the server at request time:** a build step moved
  into the server.
- **No types:** errors surface only at runtime.
