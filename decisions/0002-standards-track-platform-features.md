# 0002 Standards-track platform features

## Context

The architecture explores what the web platform can do without frameworks or
build tools. Several decisions rest on features that not every browser ships
yet. Broad browser support is not the priority at this stage.

## Decision

**The app uses a platform feature once it is on the standards track and
expected to reach Baseline soon. It does not wait for every browser.**

A feature qualifies when:

- it is specified by the WHATWG or the W3C,
- it ships in a stable release of at least one browser engine, without a
  flag or origin trial, and
- the other engines implement it or have signalled support.

Baseline, as published by the `web-features` project, is the measure. Every
feature the app relies on is recorded with its Baseline status and the date
it was checked.

Browsers without a feature get the page without it. The app writes no
polyfills and no fallbacks.

## Rationale

- **The goal is the architecture the platform enables next.** Designing
  around features that are about to land would bake in workarounds that
  outlive the gap.
- **The standards track keeps the bet safe.** A specified feature with
  support from several engines is expected everywhere. A single vendor's
  experiment is not.
- **No fallback code.** Polyfills and fallbacks exist only until browsers
  catch up, and they hide which constraints are real.
- **Baseline is a shared measure.** Anyone can check a feature's status in
  the same data.

## Consequences

- Some browsers get a lesser page. As of `web-features` 3.43.0 (checked
  2026-10-18), `@scope` is Baseline since 2026-03. Render-blocking scripts
  and cross-document view transitions ship in Chromium and Safari, not
  Firefox. Firefox users may briefly see unstyled components and navigate
  without transitions.
- The policy suits an exploration, not a production app with broad audience
  requirements.
- Recorded statuses go stale and are checked again before relying on them.
- Features that only exist in one vendor's plans, behind a flag or in an
  origin trial are not used.

## Alternatives

- **Baseline widely available only:** excludes `@scope`, view transitions
  and render-blocking scripts. Most of this architecture could not exist.
- **Baseline newly available only:** still excludes render-blocking scripts
  and cross-document view transitions.
- **Polyfills and fallbacks:** code that lives only until browsers catch up.
- **Any shipped feature:** risks building on features that never reach the
  other engines.
