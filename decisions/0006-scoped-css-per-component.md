# 0006 Scoped CSS per component

## Context

App CSS has to work without a build step and be scoped per app component.
App components and pages live in the light DOM
([0005](0005-shadow-dom-for-the-design-system.md)).
It should sit close to the markup it styles and load fast. There should be
only one way to write it.

## Decision

**A component's CSS file styles that component's markup and nothing else.**

When you write a component, you use plain, short class names and don't think
about any other component, whether above it or nested inside it. Styles stop
where a nested component begins, and nothing from outside reaches into
yours. A parent may touch only the outer box of a nested component, to place
it. The browser enforces this boundary, so it needs no build step and no
naming convention for classes. It only needs a marker on every root.

Every component and every page has one CSS file next to its template. The
file holds one block. It is the same for every file except for the layer and
the name:

```css
@layer components {
  @scope (.card) to (:scope [data-c] > *) { … }
}
```

`.card` names the component. `data-c` on every component and page root marks
where one ends and the next begins.

All CSS sits in cascade layers, ordered once for the whole app:

- **`design-system`**: lowest priority. The design system's global styles are
  imported into it.
- **`base`**: document-level rules.
- **`components`**: one block per component, as above.
- **`pages`**: one block per page. A page is scoped like a component, so it
  wins over components only on the roots of the components it places.

The first stylesheet declares the order and imports the design system:

```css
@layer design-system, base, components, pages;
@import url(…/design-system.css) layer(design-system);
```

A higher layer always wins, whatever the specificity or load order of the
rules. The app builds on the design system's styles and tokens and can adjust
them freely from its own layers. A design-system component's inside is
behind its shadow root, and the app places it like a nested app component.

The server links the files a page uses in `<head>`, at versioned URLs over
HTTP/2 ([0003](0003-no-build-step.md)).

## Rationale

- **The platform does the scoping.** `@scope` replaces naming conventions and
  build-time class renaming with a boundary the browser itself enforces.
- **Scoped, not isolated.** Inherited properties and global styles still
  reach into a component. `@scope` limits where selectors match. It does not
  wall the component off the way shadow DOM does.
- **Layers make priority independent of load order.** Each page links only
  the files it uses, so the order of rules changes from page to page. Without
  layers, specificity and that changing order would decide which rule wins.
  With layers, the role of a file decides: the design system, the app base,
  a component or a page.
- **Layer order follows first appearance.** A design system that ships its
  own layers would rank above or below the app depending on which file the
  browser reads first. Importing it into `design-system` nests its layers
  there, so they rank lowest on every page.
- **Flat selectors inside a layer.** Within one layer, specificity is
  compared before scope proximity, so a more specific parent rule wins on a
  child component's root. Proximity only breaks ties, in the child's favour.
  Selectors therefore stay flat, and overrides go into a higher layer instead
  of a more specific selector.
- **Inline styles don't scale.** A `<style>` in every component instance
  slowed rendering more than linearly in every browser, by up to 60× on long
  lists. Inline CSS is also re-sent with every page instead of being cached.
- **Separate files cost nothing over HTTP/2.** One file per component loaded
  as fast as a single bundle. Each file is cached on its own, and a page only
  loads what it uses.
- **Stylesheets belong in `<head>`.** Placed in the body, they change the DOM
  structure that selectors depend on, and the page paints in parts.

## Consequences

- Colocation means a file next to the template, not CSS inside the markup.
- All app CSS lives in a layer. Unlayered CSS would beat every layer, and
  `!important` reverses the layer order, so neither is used.
- The design system's styles load through `@import`, which the browser
  fetches only after the file that contains it.
- A root without `data-c` silently lets its parent's styles in, and the
  browser does not enforce flat selectors. Both need an automated check.
- The server knows a page's components before it renders `<head>`, as it
  does for design-system component scripts
  ([0007](0007-design-system-components-before-first-paint.md)).
- Component names are global class names. Components leave their outer
  spacing and placement to the parent.
- The boundary follows the rendered page, not the template file. Markup
  passed into an app component is styled by the component that receives it.
  Markup slotted into a design-system component stays with the app
  component that wrote it.

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
