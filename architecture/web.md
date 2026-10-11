# Vanilla web architecture

First iteration, 2026-10-11. Describes the direction we settled on for building
web user interfaces and for the `vanilla-web` skill that teaches it. Evidence
for the decisions lives in [`prototypes/`](../prototypes).

## Idea

User interfaces should be native to their platform. On the web that means the
platform itself is the framework: documents, links, forms, CSS and custom
elements. No client-side rendering library, no build step.

Agents know the individual web specifications well but not how to combine
them into a modern, framework-free application. They fall back to SPA habits or
legacy patterns. The skill exists to supply that missing architectural layer.

## The architecture

### Multi-page app

- Every URL returns a complete HTML document rendered on the server.
- Navigation is real navigation. The URL, links and forms hold the state.
- Mutations use POST, then redirect, then GET. Validation errors re-render the
  form with the submitted values.
- Tabs, filters and similar view state are URLs where it makes sense, so deep
  links and the back button work for free.

### Smoothness from the platform

- Cross-document view transitions (`@view-transition { navigation: auto; }`)
  make navigations feel continuous.
- Speculation rules prerender likely next pages as a speed-up.
- The back/forward cache restores pages instantly. Transient UI such as open
  drawers or dialogs is closed before leaving a page.
- While a new page loads, the browser keeps the old one on screen, so there is
  no white flash between pages.

### No build tools

- Plain HTML, CSS and ES modules, loaded from stable, versioned URLs.
- Modern CSS: cascade layers, `@scope`, nesting, custom properties, container
  queries, `:has()`.
- Third-party code is loaded as published ES modules from a pinned CDN URL or
  copied into the project.

### Design system

- A design system of web components is a precondition, not something the app
  builds. Web Awesome is the reference implementation.
- The design system gives consistency through encapsulation: components expose
  attributes, slots, parts and tokens, and app code does not style their
  internals.
- The app uses native elements for document structure (`main`, `nav`,
  `article`, headings, links) and design-system components for visible UI.
- Which design system a project uses, its element prefix and its rules are
  declared in the project brief (`AGENTS.md` or `DESIGN.md`). If there is none,
  the agent asks which one to use instead of improvising components.

### Where JavaScript goes

Each step is only taken if the previous one cannot do the job:

1. Semantic HTML
2. CSS
3. Declarative platform features (`<dialog>`, `popover`, invokers, `<details>`)
4. A design-system component
5. A small module script for page-level behaviour
6. An island: a light-DOM custom element for app-specific behaviour

Islands enhance server-rendered markup. They read their configuration from
attributes, communicate through events and keep cross-page state in the URL or
in storage, not in memory.

### First frame is final

The central UX rule: a page paints its final layout in the first frame. No
layout shifts, no unstyled flashes.

- Each page declares the components it uses as render-blocking module scripts
  (`<script type="module" blocking="render" src="…">`), generated from one
  server-side helper with the pinned version. Shared code is cached across
  pages.
- No autoloader for visible components.
- The server renders all component state as attributes (`active`, `value`,
  `checked`, errors), so nothing visible is decided after the first paint.
- Layout comes from CSS. A component that measures itself and changes the
  layout after the first paint is not used until it passes the first-frame test.

## Accepted trade-offs

- The first visit is slower than a native-only page because component
  definitions block rendering. Later pages reuse the cache.
- Components need JavaScript. Pages without JavaScript are not a goal.
- Firefox currently lacks render-blocking scripts and cross-document view
  transitions. Pages work there but may briefly show unstyled components.

## The skill

### Shape

One skill, `vanilla-web`, for authoring apps. It is self-contained, as
skills.sh installs skills individually. Later siblings follow the pattern
`vanilla-<platform>` (`vanilla-android`, `vanilla-ios`, …). Authoring a design
system is a separate skill (`vanilla-web-design-system`). Library-specific rules
go into small adapter skills (`vanilla-web-awesome`) only where general rules are
not enough.

### Contents

`SKILL.md` holds what prevents drift on every task:

- preflight: find or choose the design system
- the mental model and the JavaScript ladder
- one canonical file structure
- an anti-pattern table ("if you are about to …, do this instead")
- the first-frame rule
- the server contract
- an index of references with "read when …" triggers

`references/` holds one file per pattern (view transitions, speculation rules,
islands, CSS architecture, forms, state, design-system consumption,
performance). Older specs are not explained. The newest features come with
exact syntax and a verification date.

`assets/` holds a reference skeleton (layout, base CSS, an example island).

`scripts/` holds deterministic checks: components used but not declared, an
autoloader in use, styling of component internals, and a first-frame test for
new components.

### Composition with other skills

- Backend skills: this skill only assumes a server contract. Every URL renders
  a full document from a shared layout, mutations follow
  POST/redirect/GET, and static assets are served with long-lived caching.
  Routing, templating and data belong to the backend.
- Design-system skills: they own components, tokens and visual composition.
  This skill owns documents, navigation, transitions, islands, state, forms
  and loading. Conflicts are settled in the project brief, which both sides
  read first.

## Alternatives considered

- SPA with a rendering library: what the architecture replaces.
- Native elements styled only by the design system's CSS: fewer rules and no
  JavaScript cost, but consistency by convention instead of encapsulation.
  Remains the fallback where a component fails the first-frame test.
- Tailwind: requires a build step for production use and gives a toolkit
  rather than a design system.

## Open points

- Upstream fixes for `<wa-page>` and `<wa-tab-group>` (first frame) and for
  clearing server-rendered `custom-error`. Workarounds exist until then.
- Components that may measure themselves are untested: textarea, carousel,
  scroller, split-panel, stepper.
- Browser support policy and canonical file structure need to be written down.
- Evals: run realistic tasks with and without the skill to find where agents
  drift, and write the skill from those failures.
