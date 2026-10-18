# 0001 CSS authoring

## Context

App CSS has to work without a build step and be scoped per component.
It should sit close to the markup it styles and load fast. There should be
only one way to write it.

## Decision

**A component's CSS file styles that component's markup and nothing else.**

When you write a component, you use plain, short class names and don't think
about any other component, whether above it or nested inside it. Styles stop
where a nested component begins, and nothing from outside reaches into
yours. A parent may touch only the outer box of a nested component, to place
it. The browser enforces this boundary, so it needs no build step and no
naming convention.

Every component and every page has one CSS file next to its template. The
file holds one block. It is the same for every component except for the name:

```css
@layer components {
  @scope (.card) to (:scope [data-c] > *) { … }
}
```

`.card` names the component. `data-c` on every component root marks where
one component ends and the next begins.

The server links the files a page uses in `<head>`. The URLs are versioned
and cached long-term, and the files are served over HTTP/2.

## Rationale

- **The platform does the scoping.** `@scope` replaces naming conventions and
  build-time class renaming with a boundary the browser itself enforces.
- **Scoped, not isolated.** Inherited properties and global styles still
  reach into a component. `@scope` limits where selectors match. It does not
  wall the component off the way shadow DOM does.
- **Flat selectors, layers for overrides.** Specificity is compared before
  scope proximity, so on a child component's root a more specific parent rule
  wins. Proximity only breaks ties. Selectors therefore stay flat. Pages that
  must override a component do so from a higher layer:
  `base` < `components` < `pages`.
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
- HTTP/2 is part of the server contract.
- The server knows a page's components before it renders `<head>`. It already
  does for component scripts.
- Component names are global class names. Components leave their outer
  spacing and placement to the parent.
- The boundary follows the rendered page, not the template file. Markup
  passed into a component is styled by the component that receives it, not
  by the one that wrote it.

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
- **Shadow DOM or CSS module scripts:** conflict with light-DOM islands or
  need JavaScript before the first paint.
- **Tailwind:** needs a build step.
