# Open questions

Resolve these progressively in documentation before beginning implementation.

## Technology and deployment

- Which launcher framework and build tool, if any?
- Which GitHub account or organisation, repository name, visibility and domain?
- Which source branch publishes the launcher and each experience?
- Which iframe origins, sandbox settings and hardware permissions are appropriate?
- Is PWA installation part of the first delivery? What offline behaviour is actually required?

## Theme contract

- Which initial themes should be implemented? Compare sketches before selecting visual design.
- What are the exact module hooks, action injection and state subscription signatures?
- Which storage mechanism should retain the device-local theme preference? Precedence, persistence intent, reset and fallback behaviour are confirmed in [theme preferences](specification.md#theme-preferences).
- What viewport rule selects compact versus wide, and when should automatic theme selection be reevaluated on the home screen? Theme replacement during play is ruled out by the confirmed [theme-switching policy](specification.md#theme-selection-during-play).

## Display API

- Validate schema fields, request IDs, availability notification timing and legacy defaults.
- Define who may change declarations dynamically, if required; current draft assumes static declarations.
- Define UI treatment when one caller changes the mode while the other has a pending request.
- Validate user-activation handling and ownership when the launcher is already in native fullscreen.
- Define when integration is established: the current draft sends `launcher.init` once after iframe load but has no acknowledgement or retry policy. Keep protocol acknowledgement distinct from the optional application-ready signal, and define when initial display availability and effective state are sent.
- Clarify request arbitration: does an invalid newer request supersede pending fullscreen activation, and how are asynchronous browser results handled after cancellation, another request or a session change?
- Clarify effective display state when native fullscreen already belongs to the launcher before an experience starts, including how explicit `theme` or `page` requests interact with that pre-existing fullscreen session.

## Applications and content

- Obtain repository/metadata URLs, owners and actual assets for the six named experiences.
- Which built-in apps are required? Camera, gallery and notes are examples only.
- Is real camera access needed, or will simulated content suffice?
- What progress should persist, and how will teams communicate that to players?
- Who supplies descriptions, categories, credits and any editorial mappings between experiences?

## Validation

- What short delay should dismiss loading feedback when no `experience.ready` arrives? The [loading policy](specification.md#loading-feedback-and-failures) is confirmed; this timing must not become a success or failure test.
- Define supported devices and browsers, accessibility target and first-version acceptance criteria.
- Exercise integration with at least one local and one genuinely cross-origin application.
- Check catalogue growth, long titles, keyboard operation, browser Back and native fullscreen exit.
