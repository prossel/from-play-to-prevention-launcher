# Decisions

Baseline consolidated on 24 November 2026. These decisions describe the discussion, not shipped functionality.

| Decision | Current direction |
| --- | --- |
| Phase | Develop and consolidate the specification before implementation. |
| Audience | French-speaking young people aged 12-16, within From Play to Prevention. |
| Languages | English development and technical documentation; French product controls and content. Preserve supplied experience titles. |
| Presentation | Modular themes; phone is optional. No graphic identity selected. |
| Devices | Responsive mobile and desktop use. |
| Theme choice | Use the configured default unless automatic selection is enabled; a valid saved manual choice takes precedence when manual selection is allowed. See [theme preferences](specification.md#theme-preferences). |
| Theme preference | Remember manual choices on the device. “Réinitialiser le thème” clears the preference; unavailable storage or an unavailable choice falls back to configured selection. |
| Theme switching | Select and switch themes at home. Keep the active theme during play; adapt it to viewport changes while preserving the iframe, and defer requested theme switches until home. See [specification](specification.md#theme-selection-during-play). |
| Launcher ownership | Separate repository, GitHub Pages deployment, optional dedicated domain. |
| Experience ownership | Independent team repositories and independently deployed static web applications. |
| Built-in apps | Launcher may host apps in subfolders using the same metadata and iframe contract. |
| Catalogue | Launcher references metadata endpoints; it does not need to merge team repositories. |
| Publication | Push to a dedicated branch such as prod can trigger each site's deployment. |
| Running apps | Iframe within the launcher; new-tab launch is only an optional fallback. |
| Loading | Non-blocking “Chargement…” with “Accueil” available. Optional `experience.ready` clears it; otherwise dismiss it after a short delay without claiming success. Identified failures offer “Réessayer” and “Accueil”. Messaging is not required to run. See [specification](specification.md#loading-feedback-and-failures). |
| Exit | “Accueil” and valid experience exit requests close immediately through the same cleanup path, without a confirmation dialog or save-before-close handshake in version 1. |
| Progress on exit | Each experience saves during play if needed; reopening creates a fresh iframe and resumes only what the experience itself saved and restores. See [specification](specification.md#leaving-is-not-pausing). |
| Navigation | Experience can request hide/show; no redundant immersive metadata capability. |
| Recovery | No additional launcher recovery control when navigation is hidden. If stuck, refresh the launcher page or close and relaunch the installed PWA to return home. |
| Unstable experiences | Keep known unstable experiences out of the launchable catalogue until fixed; no automatic stability-detection mechanism is required. |
| Display | Theme frame, full client page and native browser fullscreen are distinct modes. |
| Display support | Both theme and experience declare supported modes. |
| Initial display | Start in common `theme`, otherwise common `page`. If neither is shared, block launch and offer a compatible installed theme where available. See [API](api.md#supported-modes-and-launcher-mediation). |
| Fullscreen fallback | If fullscreen entry fails, use common `page`, otherwise common `theme`. Leaving fullscreen restores the previous common non-fullscreen layout. See [API](api.md#fullscreen-transitions-and-restoration). |
| Display control | Theme controls and experience requests are mediated by one core manager. |
| Availability | Core supplies compatible modes; theme can hide or disable unavailable controls. |
| State preservation | Changing display mode must not reload the experience. |
| Browser limits | Native fullscreen is conditional; effective state and fallback must be explicit. |
| Future interactions | Shared notes/gallery and cross-app story state are possible, not yet specified. |

API names, message envelopes, session tokens, metadata defaults and the selection configuration schema are draft technical proposals documented in their respective contracts. They can be refined without revisiting the product decisions above. Theme preference precedence, device-local persistence and reset behaviour are confirmed; the storage mechanism remains undecided. Initial display selection, the common non-fullscreen layout requirement and fullscreen failure fallback and restoration ordering are confirmed; pending-request arbitration and pre-existing launcher fullscreen ownership remain open.
