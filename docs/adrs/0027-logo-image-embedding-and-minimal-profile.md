# ADR-0027: Logo embedded as an image with a minimal v1 sanitizer profile

| Field | Value |
|---|---|
| Status | Accepted |
| Date | 2026-10-16 |
| Owner | Tech lead, Security reviewer, Product owner |
| Phase | P0.1 (decision), P0.5 (proof) |
| Related | PRD §13.6, §16.4, §16.5; technical-spec §3, §12.3, §13.2, §14; ADR-0005, ADR-0009, ADR-0013, ADR-0014, ADR-0026 |

## Context

The `flowframe-svg-logo/v1` profile (PRD §16.5) had to support gradients, clip paths, masks, internal `use`, ten filter primitives and `<style>` validated by a real CSS parser. Technical-spec §12.3 inlined the sanitized logo nodes into the final SVG, which required deterministic renaming of colliding IDs and rewriting of every fragment reference. Together they made the sanitizer the largest security-sensitive component of v0.1 and made the P0.5 spike long.

Two properties of the problem allow a smaller design:

- An SVG document referenced through an `<image>` element is rendered by browsers as a separate static document: scripts do not run and external resources are not loaded. Its IDs live in their own document, so they cannot collide with the body. D2 already embeds the generic icons this way (ADR-0013).
- Logos are supplied by project maintainers once per project, not by arbitrary authors per diagram. Asking for a simple export is a reasonable one-time cost.

Common editors differ in how they store styling. Inkscape writes `style="fill:…;stroke:…"` attributes by default and has no simple switch to presentation attributes. Illustrator can export either presentation attributes or a `<style>` element with classes. A profile that rejected every `style` attribute would reject most Inkscape files.

## Alternatives considered

- **Keep the full profile and inline logo nodes.** Rich logos are accepted as is. Costs the stylesheet validator, the filter and mask allowlists, ID rewriting and the larger fixture corpus.
- **Embed the sanitized logo as a separate image document, and start v1 with a minimal profile.** Adopted.
- **Minimal profile without `style` attributes.** Smallest validator, but rejects default Inkscape output and pushes users to external conversion tools. Rejected.
- **Accept only raster logos (PNG).** Simplest to embed, but logos lose sharpness when scaled, and embedded rasters are rejected elsewhere in the profile. Rejected.

## Decision

1. The sanitized logo is embedded as a separate image document: an `<image>` element whose `href` is the `image/svg+xml` data URI of the canonical sanitized bytes, sized to the "contain" box of PRD §13.6. While ADR-0005 is in force the compositor writes this element; if ADR-0026 is accepted, the D2 image object carries the same data URI. Logo nodes are never inlined into the diagram document, so no ID rewriting or reference renaming is needed.
2. `flowframe-svg-logo/v1` accepts only:
   - structure and shapes: `svg`, `g`, `defs`, `title`, `desc`, `path`, `rect`, `circle`, `ellipse`, `line`, `polyline`, `polygon`,
   - paint servers: `linearGradient`, `radialGradient`, `stop`,
   - `clipPath` and `use` with fragment-only references (`href="#id"`, `xlink:href="#id"`, `url(#id)`),
   - presentation attributes from the allowlist in `rules/svg-logo-profile-v1.md`,
   - `style` attributes, parsed as a CSS declaration list by a real CSS parser. Each declaration must name a property equivalent to an allowed presentation attribute and carry a value valid for that attribute; `url()` is allowed only as `url(#id)`.
3. `v1` rejects `<style>` elements, `class` attributes, `mask`, `filter` and every filter primitive, in addition to everything PRD §16.5 already rejects. The diagnostic names the construct and tells the author how to export without it.
4. Defence in depth stays in place: bounded parsing with DTDs, entities and network disabled; fail-closed allowlist; the ADR-0009 limits; deterministic canonical output; and manifest hashes.
5. The final SVG verifier accepts the embedded logo only when its decoded bytes equal the sanitized bytes recorded in the manifest (`sanitizedSha256`).
6. Masks, filters and stylesheets can be added later only through a new profile version (`flowframe-svg-logo/v2`) with its own ADR.
7. The generic icon pack (ADR-0013) uses the same `v1` profile.

## Consequences

- Removed from v0.1: stylesheet (selector) validation for logos, the mask and filter allowlists, the filter-region limit of ADR-0009, and logo ID rewriting with reference renaming.
- `tinycss2` stays: it parses `style` attribute declaration lists here and renderer CSS for the final-output allowlist of ADR-0008.
- The P0.5 corpus shrinks to the accepted subset, the rejection cases and the limits.
- Logos that use `<style>`, classes, masks or filters must be re-exported. This is a documented, one-time action for the project maintainer.
- The logo `<title>` of ADR-0014 item 6 is attached to the `<image>` element, which avoids the cross-document references that inlining needed.
- Non-browser consumers in ADR-0022 may treat embedded SVG images differently. That is part of the verification below, not an assumption.
- Documents changed together with this ADR: PRD §13.6, §16.5; technical-spec §3, §12.3, §13.2, §14, §18, §19; implementation-plan P0.5, P4.4, P4.5, P4.10; amendment notes in ADR-0005, ADR-0009, ADR-0013 and ADR-0014.

## Verification

- P0.5: representative logos exported from Inkscape (default `style` attributes) and from Illustrator (presentation attributes) are accepted and render identically in every ADR-0022 consumer when embedded as an image.
- Forbidden constructs (`<style>`, `class`, `mask`, filters, scripts, external references, `foreignObject`, rasters) fail closed with actionable diagnostics.
- `style` declarations with a disallowed property, an invalid value or a non-fragment `url()` fail closed.
- Repeat sanitization is byte-identical, and so are the manifest input and output hashes.
- An embedded logo with IDs equal to body IDs renders correctly without rewriting.
- A final SVG whose embedded logo bytes differ from `sanitizedSha256` fails verification.
