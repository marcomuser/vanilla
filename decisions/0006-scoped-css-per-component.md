# 0006 Scoped CSS per component

## Context

App components and pages live in the light DOM
([0005](0005-shadow-dom-for-the-design-system.md)) and are served without a
build step ([0003](0003-no-build-step.md)). Their CSS needs a boundary per
component. It should sit close to the markup it styles and load fast. There
should be only one way to write it.

## Decision

**A component's CSS file styles that component's markup and nothing else.**

Every component and every page has one CSS file next to its template. The
file holds one block, the same in every file except for the layer and the
name:

```css
@layer components {
  @scope (.card) to (:scope [data-c] > *) { … }
}
```

`.card` names the component. `data-c` on every component and page root marks
where one ends and the next begins. Styles stop there: a parent reaches only
the root of a nested component, to place it, and nothing from outside reaches
further in. Class names stay plain and short.

All CSS sits in cascade layers. The first stylesheet declares their order
once for the whole app and imports the design system:

```css
@layer design-system, base, components, pages;
@import url(…/design-system.css) layer(design-system);
```

- **`design-system`**: the design system's global styles, lowest priority.
- **`base`**: document-level rules.
- **`components`**: one block per component.
- **`pages`**: one block per page. A page is scoped like a component, so it
  wins over components only on the roots of the components it places.

The server links the files a page uses in `<head>`, at versioned URLs
([0003](0003-no-build-step.md)).

## Rationale

- **The browser enforces the boundary.** `@scope` replaces naming
  conventions and build-time class renaming. It limits where selectors
  match but does not isolate: inherited properties and global styles still
  reach in.
- **The role of a file decides priority, not load order.** Each page links
  only the files it uses, so the order of rules changes from page to page.
  A higher layer wins whatever the specificity or order, so the app can
  adjust the design system freely from its own layers.
- **The design system ranks lowest on every page.** Layer order follows
  first appearance, so layers the design system ships would rank above or
  below the app depending on which file the browser reads first. Importing
  it into `design-system` nests them there.
- **Selectors stay flat.** Within one layer, specificity is compared before
  scope proximity, so a more specific parent rule wins on a child's root.
  Proximity only breaks ties, in the child's favour. Overrides go into a
  higher layer, not a more specific selector.
- **Inline styles don't scale.** A `<style>` in every component instance
  slowed rendering more than linearly in every browser, by up to 60× on long
  lists, and is re-sent with every page.
- **Separate files cost nothing over HTTP/2.** One file per component loaded
  as fast as a single bundle. Each is cached on its own, and a page loads
  only what it uses.
- **Stylesheets belong in `<head>`.** In the body they change the DOM
  structure that selectors depend on, and the page paints in parts.

## Consequences

- Unlayered CSS would beat every layer, and `!important` reverses the layer
  order, so neither is used.
- The design system's styles load through `@import`, which the browser
  fetches only after the file that contains it.
- A root without `data-c` silently lets its parent's styles in, and the
  browser does not enforce flat selectors. Both need an automated check.
- The server knows a page's components before it renders `<head>`, as it
  does for design-system definitions
  ([0007](0007-design-system-components-before-first-paint.md)).
- Component names are global class names. Components leave their outer
  spacing and placement to the parent.
- The boundary follows the rendered page, not the template. Markup passed
  into an app component is styled by the component that receives it.

## Alternatives

- **`<style>` in each component instance:** cost grows faster than the page,
  and the CSS can't be cached.
- **One `<style>` per page or component type:** fast, but not cached across
  pages.
- **`<link>` in `<body>` beside the component:** breaks structural selectors
  and paints the page in parts.
- **One concatenated bundle:** no faster over HTTP/2. It ships unused CSS and
  any change invalidates all of it.
- **Global CSS with naming conventions:** styles leak into nested components.
- **Shadow DOM or CSS module scripts:** turn every app component into a
  custom element with a definition and styles per shadow root
  ([0005](0005-shadow-dom-for-the-design-system.md)).
- **Tailwind:** needs a build step.
