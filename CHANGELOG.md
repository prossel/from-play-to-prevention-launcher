# Specification history

## Swiss design exploration 2026-11-24

- Added Style suisse as proposal 13 in the existing theme exploration document, with mobile/desktop descriptions and a comparison board following the existing format: sketches, direction A and direction B on mobile and desktop.

## GitHub Pages CORS guidance 2026-11-24

- Clarified that GitHub Pages normally supplies public wildcard CORS access without team configuration and does not support custom response-header configuration directly.
- Required verification of actual published HTML and optional JSON access, including custom-domain, redirect or proxy responses; limited header-configuration examples to hosts that support them.
- Updated the minimum and optional team checklists accordingly.

## Minimal catalogue and page discovery decision 2026-11-24

- Reduced catalogue entries to `id` and `url`, removing the title requirement.
- Defined display-name priority as optional JSON title, page title, then ID fallback; icon priority is JSON override, page icon links, root favicon, then generic fallback.
- Documented static HTML discovery, cross-origin CORS requirements and graceful fallback without blocking embedding or launch.
- Aligned team instructions, architecture, decisions and acceptance checks; left detailed favicon candidate selection and bounded discovery timing open.

## Experience team integration workflow 2026-11-24

- Replaced mandatory metadata-endpoint registration with published folder/page URLs; catalogue entries supply stable identity and launch URLs (title registration superseded by the minimal-catalogue decision above).
- Defined zero-code baseline integration, optional metadata and optional messaging, retaining default `theme` support and iframe compatibility requirements.
- Documented standalone development, launcher forks, integration testing and catalogue pull requests.
- Documented CORS for optional JSON separately from iframe embedding and message-origin validation, with hosting examples and troubleshooting.
- Aligned API defaults, architecture, decisions and acceptance checks; retained proposed schema/discovery details and bounded-fetch timing as open implementation choices.

## Loading feedback decision 2026-11-24

- Confirmed non-blocking “Chargement…” with “Accueil” available, cleared by optional `experience.ready` or after a short delay without claiming successful loading.
- Kept messaging optional and distinguished indicator dismissal from failure detection and protocol acknowledgement.
- Confirmed French feedback with “Réessayer” and “Accueil” for identified loading failures; aligned API, integration guidance and acceptance checks.
- Left the exact loading-indicator delay open.

## Immediate exit decision 2026-11-24

- Confirmed immediate close through the same cleanup path for “Accueil” and valid experience exit requests, without confirmation or a save-before-close handshake in version 1.
- Confirmed experience-owned saving during play and restoration on a fresh launch; the launcher does not wait for saving at exit.
- Aligned the API, team checklist and acceptance checks, and removed the resolved exit-policy question.

## Theme preference decision 2026-11-24

- Confirmed configured default or enabled automatic selection, with valid saved manual choices taking precedence when manual selection is allowed.
- Confirmed device-local preference persistence and “Réinitialiser le thème” to clear it and resume configured selection.
- Defined fallback to configured selection for unavailable storage or an unavailable choice, retaining home-only theme switching.
- Aligned the proposed architecture configuration, decisions, theme guidance and acceptance checks; left storage technology and viewport rules open.

## Theme switching decision 2026-11-24

- Confirmed theme selection and switching on the home screen, keeping the active theme throughout a running experience.
- Confirmed responsive adaptation within that theme while preserving the iframe, with requested theme switches deferred until returning home.
- Aligned architecture, theme guidance and acceptance checks; kept selection precedence, persistence, breakpoint and home-screen reevaluation timing open.

## Display defaults decision 2026-11-24

- Confirmed initial common `theme`, otherwise common `page`; block incompatible launches and offer a compatible installed theme where available.
- Confirmed fullscreen entry-failure fallback to common `page`, otherwise common `theme`, and restoration of the prior non-fullscreen layout on browser fullscreen exit.
- Clarified iframe preservation across all display transitions and aligned decisions, API status, theme guidance and acceptance checks.
- Removed the resolved display-defaults question while retaining pending-request and pre-existing fullscreen ownership questions.

## Recovery decision 2026-11-24

- Confirmed that hidden navigation needs no additional launcher recovery control; refresh the launcher or close and relaunch the installed PWA if stuck.
- Specified restart at home without automatically reopening the experience, while retaining the normal experience-owned exit requirement.
- Kept known unstable experiences out of the launchable catalogue until fixed, without adding automatic stability detection.
- Updated the API, product acceptance checks, integration and theme guidance, and removed the resolved recovery-control question.

## Workspace review 2026-11-24

- Recorded existing local Git history on `main`, no configured remote and the requirement for separate authorisation of further commits and publication.
- Clarified that metadata preserves supplied titles, including English titles, and removed an unsupplied content summary from the named experience example.
- Kept `prod` as a deployment example rather than a selected publication branch.
- Identified unresolved recovery-control, handshake, request-arbitration and pre-existing fullscreen behaviour in the open questions without confirming new protocol decisions.

## Consolidated starter 2026-11-24

- Collected the current specification into topic-based canonical Markdown documents.
- Preserved the 12-theme exploration dossier with sketches and two visual directions per theme, each on mobile and desktop.
- Preserved earlier phone sketches and graphical explorations as historical design context.
- Recorded the move from a phone launcher to interchangeable responsive themes.
- Included navigation hide/show and experience exit messages.
- Included theme/page/native fullscreen display modes, capability negotiation between theme and experience, theme-triggered changes and core mediation.
- Separated confirmed product direction from technical proposals and open questions.
- Added Codex continuation instructions and Git preparation guidance. No implementation or remote repository exists in this package.
