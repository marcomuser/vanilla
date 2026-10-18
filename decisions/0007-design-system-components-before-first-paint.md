# 0007 Design-system components defined before first paint

## Context

Every navigation runs a cross-document view transition, so every page has to
paint its final layout in the first frame
([0004](0004-view-transitions-on-every-navigation.md)). Design-system
components render
through shadow DOM ([0005](0005-shadow-dom-for-the-design-system.md)), and
they break both when they are defined after the first paint: until then the
browser renders an undefined element as an unknown inline element around its
light DOM, then re-renders it once the definition arrives.

## Decision

**Every design-system component that is visible in the first frame is
defined before the first paint.**

The page declares the definitions of those components in `<head>`, one
render-blocking module script per component, from the design system vendored
at a pinned version ([0003](0003-no-build-step.md)):

```html
<script type="module" blocking="render" src="…/button/button.js"></script>
```

The same server helper that links the page's stylesheets writes these tags,
from the design-system components the page rendered.

The criterion is how a component looks before it is defined. A
design-system component's rendering comes from its definition, so it blocks.
An island enhances server-rendered markup that already looks final, so its
definition loads without blocking.

The server renders all component state as attributes, so nothing visible is
decided after the first paint.

## Rationale

- **View transitions capture the first frame.** The transition animates to
  the new page as it is first rendered. If its components are not defined
  yet, it ends on the unfinished page and the layout changes afterwards.
  Render-blocking resources delay that first render, so the transition
  captures the final page.
- **Undefined components shift the layout.** Their shadow DOM, styles and
  layout only exist after the definition runs. Painting before that shows a
  different layout than the final one.
- **`blocking="render"` keeps the page off screen until its definitions have
  run.** Everything visible is defined in the first frame, with no hiding
  rules and no timers.
- **Only the head knows in time.** Definitions discovered after parsing, by
  scanning the DOM, arrive after the first paint.
- **Definitions are cached across pages.** The pinned version gives stable
  URLs, so the cost falls mainly on the first visit.

## Consequences

- The first visit paints later than a page of native elements. Later pages
  reuse the cached definitions.
- Pages need JavaScript for their components. Pages without JavaScript are
  not a goal.
- Browsers that ignore `blocking="render"` paint before the definitions
  arrive and may show unstyled components
  ([0002](0002-standards-track-platform-features.md)).
- The server knows a page's components before it renders `<head>`, as for
  stylesheets in [0006](0006-scoped-css-per-component.md).
- A component that measures itself and changes the layout after the first
  paint is not used until it renders its final layout in the first frame.
- Islands must render correctly without their definition.

## Alternatives

- **Autoloader:** finds components in the DOM, so definitions arrive after
  the first paint and after the view transition.
- **Hiding undefined elements (`:not(:defined)`):** avoids the unstyled
  flash, but the view transition captures the page without them and they
  appear afterwards.
- **Declarative shadow DOM:** the only other way to paint encapsulated
  components final in the first frame, and it needs no JavaScript. But the
  server has to render every component's internals, which ties the backend
  to the design system's rendering code. Each instance also carries its own
  styles, and a `<style>` per instance slowed long lists by up to 60× in
  [0006](0006-scoped-css-per-component.md).
- **Blocking every component, islands included:** delays the first paint for
  components that already look final.
- **Native elements styled by the design system's CSS:** no JavaScript cost,
  but consistency by convention instead of encapsulation. Remains the
  fallback for components that cannot render their final layout in the
  first frame.
