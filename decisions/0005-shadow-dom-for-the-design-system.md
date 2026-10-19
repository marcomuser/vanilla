# 0005 Shadow DOM for the design system, light DOM for the app

## Context

UI code comes from two places. The design system holds the design language:
the visible controls and how they look, shared across apps. The app composes
them into its own components and pages. Both need a boundary, but not the
same one. The design language has to stay consistent, and the app has to be
easy to write.

## Decision

**Design-system components encapsulate their rendering in shadow DOM. App
code lives in the light DOM.**

The app uses a design system whose components are custom elements with
shadow DOM. It changes them only through what they expose: attributes,
slots, parts and tokens.

The app writes no shadow DOM. Its code comes in two kinds:

- **App components**: server-rendered templates with a `data-c` root and a
  scoped CSS file ([0006](0006-scoped-css-per-component.md)).
- **Islands**: app components with a light-DOM custom element that adds
  behaviour to their server-rendered markup.

Two boundaries follow. The shadow root separates the app from the design
system. `@scope` separates app components from each other.

## Rationale

- **Encapsulation keeps the design language consistent.** App CSS cannot
  reach a design-system component's internals, not even by accident. The
  app adjusts the design through tokens from its own layers, and every
  component follows.
- **The host is in the light DOM.** The app places a design-system component
  through its outer box, like a nested app component.
- **The light DOM keeps app code simple.** Short class names, scoped by
  `@scope`, with parent and page rules reaching component roots. App
  components need no JavaScript and no definition.
- **Slotted content stays with the app.** Children placed in a design-system
  component remain light DOM. The host carries no `data-c`, so the app
  component around it styles them, and the design system reaches only the
  top-level slotted elements, with `::slotted`.

## Consequences

- The design system is a precondition, not something the app builds. What
  it does not expose, the app cannot change, and fixes go upstream.
- Design-system components need their definitions before they render. See
  [0007](0007-design-system-components-before-first-paint.md).
- ID references such as `aria-labelledby` do not cross a shadow root.
- Design-system form controls take part in forms only if the design system
  implements form association.

## Alternatives

- **Shadow DOM for app components too:** every app component becomes a
  custom element with a definition and its own styles per shadow root.
- **Light-DOM design-system components:** their internals sit in the app's
  markup, where app rules from higher layers win over them. Consistency
  holds by convention only.
- **Native elements styled by the design system's CSS:** no JavaScript cost,
  but consistency by convention instead of encapsulation. Remains the
  fallback for components that cannot render their final layout in the
  first frame.
