# Decisions

Baseline consolidated on 24 November 2026. These decisions describe the discussion, not shipped functionality.

| Decision | Current direction |
| --- | --- |
| Phase | Develop and consolidate the specification before implementation. |
| Audience | French-speaking young people aged 12-16, within From Play to Prevention. |
| Languages | English development and technical documentation; French product controls and content. Preserve supplied experience titles. |
| Presentation | Modular themes; phone is optional. No graphic identity selected. |
| Devices | Responsive mobile and desktop use. |
| Theme choice | Allow configured default, automatic preselection and/or manual selection; final policy remains open. |
| Launcher ownership | Separate repository, GitHub Pages deployment, optional dedicated domain. |
| Experience ownership | Independent team repositories and independently deployed static web applications. |
| Built-in apps | Launcher may host apps in subfolders using the same metadata and iframe contract. |
| Catalogue | Launcher references metadata endpoints; it does not need to merge team repositories. |
| Publication | Push to a dedicated branch such as prod can trigger each site's deployment. |
| Running apps | Iframe within the launcher; new-tab launch is only an optional fallback. |
| Exit | Launcher provides Home by default. Experience can request exit through messages. |
| Navigation | Experience can request hide/show; no redundant immersive metadata capability. |
| Display | Theme frame, full client page and native browser fullscreen are distinct modes. |
| Display support | Both theme and experience declare supported modes. |
| Display control | Theme controls and experience requests are mediated by one core manager. |
| Availability | Core supplies compatible modes; theme can hide or disable unavailable controls. |
| State preservation | Changing display mode must not reload the experience. |
| Browser limits | Native fullscreen is conditional; effective state and fallback must be explicit. |
| Future interactions | Shared notes/gallery and cross-app story state are possible, not yet specified. |

API names, message envelopes, session tokens, metadata defaults, selection configuration and exact fallback ordering are draft technical proposals documented in their respective contracts. They can be refined without revisiting the product decisions above.
