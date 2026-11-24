# Launcher specification

Baseline consolidated on 24 November 2026. This is a specification project, not an implemented launcher. Read decisions.md for confirmed direction and open-questions.md for remaining choices.

## Purpose and language

Build a modular, themeable launcher that brings together small prevention games and other interactive applications for French-speaking young people aged 12–16. Players explore situations involving digital social life, relationships, exclusion and cyberbullying through different experiences.

The launcher is an experience hub, not a single game. Each application can have its own narrative, visual style and technology.

**Development language is English:** code identifiers, filenames, API messages, technical documentation and development comments. **The product language is French:** launcher interface, application titles, descriptions, instructions, accessibility labels, errors and experience content. Use `lang="fr"` in published French pages. The English project name “From Play to Prevention” can remain the brand.

## Inspiration and product decisions

[Windows93](https://windows93.net/) inspires an explorable environment full of applications and unexpected uses. [It is as if you were doing work](https://pippinbarr.com/itisasifyouweredoingwork/) inspires familiar interface actions becoming playful experiences. Their retro appearance is not a requirement.

The agreed foundation is:

- A mobile-first modular launcher with interchangeable presentation themes.
- Each theme adapts to mobile and desktop; the fictional phone is one optional theme.
- Theme selection may use a configured default, automatic preselection, manual selection, or automatic preselection with a manual override.
- Responsive adaptation is required within each theme and is independent of theme selection.
- Select and switch themes on the home screen. Keep the active theme throughout an experience; apply any theme switch requested during play only after returning home.
- Each experience opens in an iframe inside that interface.
- The launcher owns a navigation bar outside the iframe, with an always-available Home action by default.
- An experience may request that the bar be hidden once its own exit control is ready. No immersive capability declaration is required in metadata.
- The launcher has its own GitHub repository and GitHub Pages deployment, optionally on a dedicated domain.
- Independent teams own their repositories and publish their own applications, normally through GitHub Pages.
- Applications hosted in launcher subfolders use the same integration mechanism as external experiences.
- A catalogue of metadata URLs connects the launcher to its applications.

## Layout sketches — visual design remains open

The following two French-language mockups are earlier explorations of the optional phone theme, not a required launcher layout. The product may use other themes or offer several themes. These mockups are intentionally low-fidelity, monochrome layout sketches. They illustrate content hierarchy, application placement and navigation only. Their hand-drawn appearance is a documentation convention, not a proposed product style. Colours, typography, illustrations, icon design, branding and decorative treatment remain undecided. The labels in these earlier sketches are illustrative. The current experience catalogue is: Tik Tak Tik Tak; The Secret of St Christophe; Marked; Without a Trace; Echo; Le groupe parallèle. Preserve the supplied titles even when they are in English.

### Mobile home screen

![French mobile launcher mockup](design/assets/phone-sketch-mobile-fr.png)

The interface uses the actual mobile display. Avoid adding a second simulated operating-system status bar or gesture indicator on the real device. The featured experience card and icon grid are proposed home-screen components.

### Desktop home screen

![French desktop launcher mockup](design/assets/phone-sketch-desktop-fr.png)

In this optional theme, the phone is the interaction area. The surrounding page remains neutral; any supporting content or visual treatment will be decided later. Fit the complete phone within the available height where practical; do not simply shrink text to fit a short desktop window. At constrained sizes, allow the layout to become the mobile presentation.

### Shared interaction rules

- Keep navigation in a dedicated layout row, never over important application controls.
- When the bar is hidden, the iframe expands into the released space and the experience responds to resizing.
- Use semantic buttons and links, visible keyboard focus, readable contrast and comfortably sized touch targets; aim for at least 44 × 44 CSS pixels for primary controls.
- Support touch, mouse and keyboard. Respect reduced-motion preferences.
- Do not make hover, a hidden gesture or a keyboard shortcut the only way to complete a task.
- Reserve space for real mobile safe areas and accommodate the on-screen keyboard.
- Preserve a coherent launcher identity without forcing applications to copy its visual style.

## Theme preferences

Use the configured default theme unless automatic selection is enabled. Automatic selection uses the configured viewport mapping, falling back to the default when no valid mapping is available.

When manual selection is allowed, remember the player's chosen theme on that device for later visits. A valid saved choice takes precedence over automatic selection. Offer “Réinitialiser le thème” to clear that preference and return to configured selection: automatic when enabled, otherwise the default.

If preference storage is unavailable or the chosen theme is no longer available, fall back to configured selection. Validate choices against the installed and enabled themes. These rules do not choose a storage technology, initial theme or viewport breakpoint.

## Theme selection during play

The selected theme stays active for the entire running experience. Viewport changes, including resizing and rotation, adapt that theme's responsive layout without replacing the theme or reloading, recreating or navigating the iframe. Any requested theme switch is deferred until the player returns home; it does not close the experience automatically.

Theme switching is separate from changing display mode within the active theme. The core continues to mediate display-mode requests through [api.md](api.md). The [theme preference rules](#theme-preferences), including resetting a preference, apply theme changes only at home. The compact/wide rule and automatic reevaluation timing at home remain open.

## Opening, running and leaving an application

### Default lifecycle

1. The player selects an icon.
2. The launcher records which icon opened the experience, ensures navigation is visible and displays loading feedback.
3. It creates a new iframe using the resolved entry URL and an accessible French title.
4. The experience runs within the available experience area.
5. The player activates the launcher's “Accueil” button, or the experience requests exit.
6. The launcher removes the iframe, clears session-specific listeners and state, restores navigation and shows the home screen.
7. Keyboard focus returns to the initiating icon when it is still present.

Remove the iframe rather than merely covering or hiding it; otherwise its sounds and execution can continue. An iframe being removed does not erase previously saved data or uninstall a service worker registered by the application.

Only the active iframe may control launcher navigation. Reset navigation to visible for every new launch and whenever an iframe is reloaded or navigates to a new document. Applications should normally keep internal navigation within the same document; reloading requires a new integration handshake.

### Leaving is not pausing

In version 1, activating “Accueil” or receiving a valid experience exit request closes the running application immediately through the same cleanup path. There is no confirmation dialog or save-before-close handshake, and the launcher does not wait for application saving before removing the iframe.

Opening the application again creates a fresh iframe. Progress resumes only from data the application itself saved and can restore. The launcher cannot automatically capture arbitrary application state. Applications that need progress persistence must save during play rather than rely on asynchronous work at exit. Unsaved in-memory progress is lost on close.

A save-before-close handshake is outside version 1; introducing it later would require a separate agreed protocol and timeout policy.

### Recovery from a stuck experience

No additional launcher recovery control is required when navigation is hidden. If something becomes stuck, refresh the launcher page or close and relaunch the installed PWA. The launcher starts at home with navigation visible and no running experience; it does not automatically reopen the stuck application. Unsaved in-memory progress is lost; previously saved data remains the application's responsibility.

Known unstable experiences are kept out of the launchable catalogue until fixed. This is a catalogue-maintenance decision, not an automatic stability-detection feature. The normal exit-control requirements remain in [api.md](api.md).

### New tabs and browser Back

An iframe is the normal launch mode. Opening a new tab can be an explicitly labelled fallback for an incompatible external project; it cannot preserve the launcher wrapper or guarantee an automatic return.

The launcher provides its own Home action and never relies on `window.close()`, the browser Back button or `history.back()` inside an experience. Browser Back behaviour needs testing because iframe navigations can participate in session history. Any shell routing should be designed separately from the message contract.

## PWA, viewport and fullscreen

The launcher may be installable as a PWA. Start with a web app manifest requesting `display: "standalone"`; actual presentation depends on browser and platform support. Provide an appropriate application name, icons, start URL, scope and theme colours for the chosen deployment path.

| Mode | What it changes |
| --- | --- |
| Regular mobile browser | Launcher fills the available viewport; browser controls may remain |
| Installed PWA | Can remove normal browser navigation chrome, depending on platform |
| Navigation hidden | Hides only the launcher's navigation bar |
| API `page` mode | Uses the whole launcher client area without a decorative theme frame |
| Browser Fullscreen API | Separate user-initiated enhancement, subject to browser support |

Implement native fullscreen through the display API in [display and navigation API](api.md), using the launcher root containing both the experience area and launcher controls. Do not require fullscreen for core use. Support is negotiated between theme and experience. When `page` or `fullscreen` is common, the theme must honour expansion; a phone outline is not a mandatory constraint on desktop.

The [initial display rules](api.md#supported-modes-and-launcher-mediation) and [fullscreen failure fallback and restoration rules](api.md#fullscreen-transitions-and-restoration) are confirmed requirements. All display transitions preserve the running iframe and remain independent of navigation visibility.

Use responsive viewport units and safe-area handling as appropriate. Test mobile browser toolbar changes, rotation and the software keyboard. An installed launcher does not automatically make external experiences available offline. Offline packaging and update coordination remain separate work.

## State, persistence and future interactions

Version 1 gives each application responsibility for its own state. Locally saved progress must handle unavailable or restricted storage gracefully. Applications sharing an origin should namespace keys using their stable ID; storage and permissions are not separated by repository path.

Cross-application continuity is a future extension: an experience might add a fictional image to Gallery or create a note in Notes. The launcher could mediate these actions through a separately defined data and message contract. No shared gallery, shared storage API, automatic pause/resume, completion dashboard or account system is currently specified.

## Launcher acceptance checks

| Scenario | Expected result |
| --- | --- |
| Built-in and external apps appear together | One consistent catalogue and opening behaviour |
| App with no integration code | Runs in a common non-fullscreen mode with navigation visible; absent metadata support defaults to `theme` |
| Theme and experience advertise different support | Controls reflect the intersection and browser availability |
| Launch with a common non-fullscreen layout | Start in common `theme`, otherwise common `page`; never require fullscreen to launch |
| Theme button changes mode | Core validates it; theme and experience receive the same effective state |
| Experience requests a mode outside the intersection | Error reported; effective mode preserved |
| No common non-fullscreen layout | Launch blocked with French feedback and a compatible-theme option where available |
| Home while sound is playing | Iframe is removed and its document playback stops |
| Player activates “Accueil” or experience sends a valid exit request | Same immediate cleanup and return home, without a confirmation dialog or waiting for a save handshake |
| Experience is reopened after exit | Fresh iframe; progress resumes only from data saved and restored by the experience itself |
| Immersive request after exit button is ready | Bar hides and content resizes |
| App requests show or exit | Bar returns or home opens as requested |
| Next application opens after immersive use | Navigation starts visible |
| Experience is stuck with navigation hidden | Refreshing the launcher, or closing and relaunching the installed PWA, returns home without reopening the experience; no extra recovery control is required |
| Experience is known to be unstable | It is kept out of the launchable catalogue until fixed |
| Message from another window, origin or old session | Ignored |
| Experience requests `page` | Decorative frame removed; client area used; iframe session and navigation visibility preserved |
| Experience requests `theme` | Active theme layout restored without restarting the application |
| Fullscreen request lacks user activation | Common non-fullscreen layout retained, `pending` reported and accessible activation control offered |
| Native fullscreen unsupported or rejected | Common `page`, otherwise common `theme`, remains usable; explicit fallback notification sent |
| User exits native fullscreen | Prior non-fullscreen layout restored and state notification sent |
| Any display transition, including fullscreen entry, failure or exit | Existing iframe document and application state preserved; navigation visibility unchanged |
| Experience exits after requesting fullscreen | Experience-owned fullscreen and pending requests cleaned up |
| One manifest or icon is unavailable | Other experiences remain usable |
| Embedded page is blocked or never becomes ready | Navigation remains visible unless the experience has validly requested hiding it; loading feedback does not block Home; iframe load alone is not proof of success |
| Team deploys an update | Available on a later launch after normal cache revalidation; no live interruption |
| Mobile keyboard or orientation changes | Controls and exit remain reachable |
| Desktop viewport is short or narrow | Launcher remains usable; active theme adapts |
| Viewport changes during an experience | Same theme and iframe remain active; responsive layout adapts without restarting the experience |
| Theme switch requested during play | Current experience continues in the same theme; switch applies only after returning home |
| No saved choice, or manual selection disabled | Configured automatic selection applies when enabled; otherwise use the default theme |
| Player returns with a valid saved choice and manual selection allowed | Remembered theme takes precedence over automatic selection |
| Player activates “Réinitialiser le thème” | Saved preference is cleared; configured selection resumes, with any theme change deferred until home |
| Preference storage unavailable or saved theme no longer available | Launcher remains usable through configured selection: automatic when enabled, otherwise default |
| App is opened directly | Standalone interface still works |

Test representative iOS Safari, Android Chrome and desktop browsers, in browser mode and installed mode where supported. Include at least one local app and one genuinely cross-origin app. Test iframe errors explicitly: a load event alone is not reliable evidence that an application successfully rendered. Apps without a ready signal need non-blocking loading feedback and an accessible way home.

## Remaining design choices

Before implementation, choose the launcher stack, deployment organisation/domain, initial themes, theme contract implementation, compact/wide viewport rule, final visual identity, and iframe trust/sandbox policy. These do not change the agreed catalogue-and-iframe architecture. Real hardware features, offline experiences, shared data and save-before-close require additional contracts if introduced.

## Related contracts

- [Architecture](architecture.md)
- [API](api.md)
- [Experience integration](experience-integration.md)
- [Theme development](theme-development.md)
- [Technical references](references.md)
