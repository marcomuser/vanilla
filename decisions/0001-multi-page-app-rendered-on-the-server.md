# 0001 Multi-page app rendered on the server

## Context

A web app needs routing, state, data loading and rendering. The common answer
is a single-page app, where a client-side library renders the UI and takes
over navigation. The platform already provides documents, links, forms and
history, and browsers now move between documents smoothly.

## Decision

**Every URL returns a complete HTML document rendered on the server.
Navigation is real navigation.**

- State lives in the URL, in links and in forms. Tabs, filters and similar
  view state are URLs where it makes sense.
- Mutations are form POSTs, followed by a redirect to a GET. Validation
  errors re-render the form with the submitted values.
- Every page renders from a shared layout. Routing, templating and data
  belong to the backend, in any language.
- JavaScript adds behaviour to server-rendered markup. It does not render
  pages.

## Rationale

- **The platform already does it.** History, the back button, deep links,
  loading indication and form submission work without code.
- **One place renders.** The server owns data and markup. There is no client
  router, no client state store and no hydration.
- **Navigation between documents is smooth now.** Cross-document view
  transitions ([0004](0004-view-transitions-on-every-navigation.md)), the
  back/forward cache and prerendering make it continuous and fast.
- **Every landing page is a GET.** Reload and back never resubmit a form, and
  every page can be cached, prerendered and restored.

## Consequences

- Every navigation loads a document. Speed comes from server response time,
  caching and prerendering.
- Nothing survives a navigation in memory. Client state that spans pages
  lives in the URL or in storage.
- GET requests have no side effects.
- The backend meets a contract: a full document for every URL,
  POST/redirect/GET, and versioned assets over HTTP/2
  ([0003](0003-no-build-step.md)).
- Updates without navigation are islands
  ([0005](0005-shadow-dom-for-the-design-system.md)).

## Alternatives

- **Single-page app with a rendering library:** renders on the client what
  the server already has, reimplements navigation and history, and needs a
  build step.
- **Server rendering with client hydration:** two renderers for the same UI,
  and a build step.
- **Fragment swapping (htmx, Turbo):** a script takes over navigation, which
  the browser now animates between documents by itself.
