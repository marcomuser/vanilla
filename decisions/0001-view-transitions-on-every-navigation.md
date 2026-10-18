# 0001 View transitions on every navigation

## Context

The app is a multi-page app. Every navigation loads a new document rendered
on the server. Navigations should still feel continuous, without client-side
rendering.

## Decision

**Every same-origin navigation runs a cross-document view transition.**

The app turns them on once, in the `base` layer, and turns them off for
users who prefer reduced motion:

```css
@layer base {
  @view-transition {
    navigation: auto;
  }

  @media (prefers-reduced-motion: reduce) {
    @view-transition {
      navigation: none;
    }
  }
}
```

Every page is built so that the transition captures it final: the old page
as the user left it, the new page in its final layout.

## Rationale

- **Continuity without a client-side router.** The browser animates between
  two server-rendered documents. Real navigation, URLs and the back button
  stay as they are.
- **Declarative.** One CSS rule, no JavaScript. Browsers without support
  navigate as before.
- **A transition shows both frames.** It animates from the old page's last
  frame to the new page's first. Whatever is unfinished in either frame
  becomes part of the animation.

## Consequences

- Every page renders its final layout in the first frame:
  - stylesheets in `<head>` ([0003](0003-scoped-css-per-component.md))
  - design-system components defined before the first paint
    ([0004](0004-design-system-components-before-first-paint.md))
  - all visible state rendered by the server as attributes
  - no component that measures itself and changes the layout afterwards
- Transient UI such as open dialogs and drawers is closed before leaving a
  page.
- A `view-transition-name` must be unique on a page. Names live in page
  files, which reach the roots of the components they place.
- Users who prefer reduced motion navigate without animation.
- In browsers without support, navigation works without animation. The
  first-frame rule still prevents layout shifts there.

## Alternatives

- **Client-side router with same-document view transitions:** needs
  client-side rendering, which the architecture replaces.
- **No transitions:** the browser swaps pages without continuity.
- **Transitions only between selected pages:** every page that can be a
  target still has to be final in its first frame, so opting out saves
  nothing and navigation feels inconsistent.
