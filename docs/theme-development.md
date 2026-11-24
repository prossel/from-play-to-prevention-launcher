# Theme development guide

## Scope

A theme is a trusted launcher module defining catalogue presentation, the experience container and the launcher controls. Themes are independent of applications and their repositories. A phone, desktop, library, map or other metaphor is possible; none is selected.

See [architecture.md](architecture.md) for the proposed source layout and [api.md](api.md) for the authoritative display and navigation contract. This page is an authoring guide, not a second API definition.

## Theme responsibilities

- Provide a responsive home screen from the catalogue, using the exact experience titles.
- Expose a mount area around the existing core-owned iframe rather than creating another iframe.
- Render launcher controls using core state; route actions back to the injected core actions.
- Declare supported display modes in the theme descriptor.
- Hide or disable controls according to core-computed availability, with accessible French explanations.
- Subscribe to actual display changes, including changes requested by the experience and native fullscreen exits.
- Provide the expanded layout for page or fullscreen only when declaring support for it. A phone outline or aspect constraint cannot remain compulsory in those modes.
- Preserve focus, iframe state and a usable route home during layout changes.
- Follow the [navigation and recovery policy](api.md#optional-immersive-mode): no additional recovery control is required when navigation is hidden.
- Dispose theme listeners and isolate visual styles when leaving the theme.

## Selection

Architecture must allow a configured theme, automatic mobile/desktop preselection, manual selection, or automatic preselection with manual override. These are selection policies, not separate responsive layouts. Each installed theme must work across viewport sizes. The exact policy, breakpoint and persistence behaviour remain to be decided.

Prefer applying selection on the home screen. Do not replace or restart an active experience just because a viewport changes.

## Display mediation

Theme controls and experience messages use the same core display manager. Neither caller overrides the other's support declaration. The launcher computes the intersection, applies browser constraints and reports the effective mode to both parties. Unsupported requests preserve the current layout.

The detailed common non-fullscreen launch requirement and fallback order in api.md are proposed baseline behaviours for review. If an alternative is adopted, update the one authoritative API document and its corresponding acceptance checks.

## Exploration assets

[Theme proposals](design/theme-proposals.md) include 12 themes, each with sketch, direction A and direction B, all shown on mobile and desktop. These are generated exploratory visuals. Their controls, categories, illustrations and relationships are not implementation commitments.

The earlier phone sketches and graphical mockups are kept as historical context. Their colours, device framing and example labels do not define current product requirements.
