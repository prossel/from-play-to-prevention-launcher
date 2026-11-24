# Launcher architecture

The structure below describes future implementation. Only documentation exists in this starter folder. The technology stack remains open.

## Architecture and responsibilities

| Component | Responsibility | Deployment |
| --- | --- | --- |
| Launcher core | Catalogue, iframe lifecycle, navigation state and message validation | Launcher repository and Pages site |
| Theme system | Theme selection, home layout, experience frame and navigation rendering | Launcher repository and Pages site |
| Built-in applications | Camera-inspired app, gallery, notes and other fictional phone tools | Subfolders of the launcher deployment |
| Independent experiences | Self-contained games or interactive pages | Each team's repository and published site |
| Application metadata | Identity, icon, French description and entry URL | Published alongside each application |
| Optional PWA layer | Installation and shell caching | Launcher origin |

Built-in tools can be repurposed for narrative or playful prevention activities. A gallery may contain fictional images; notes may reveal a story; a camera-like interface may offer a simulated activity. Real hardware access is not required by the concept.

Use the same loading code for local and external applications. Hosting location must not change how a player opens or exits them. The first version runs one application at a time.

### Suggested launcher repository layout

This is a proposed source organisation, not an implemented scaffold. File extensions and the build tool remain illustrative; no framework is selected. English names are used for development and French for displayed content.

```text
launcher/
  index.html
  src/
    main.js
    core/
      catalog.js
      metadata.js
      experience-host.js
      navigation.js
      display.js
      messaging.js
      session.js
    theme-system/
      registry.js
      selection.js
      theme-contract.js
    themes/
      library/
        index.js
        home.js
        frame.js
        navigation.js
        styles.css
        assets/
      phone/
        index.js
        home.js
        frame.js
        navigation.js
        styles.css
        assets/
    shared/
      ui/
      styles/
        base.css
        accessibility.css
      i18n/
        fr.json
    pwa/
      register.js
      service-worker.js
  public/
    catalog.json
    launcher.json
    manifest.webmanifest
    icons/
    apps/
      camera/
        experience.json
        index.html
        icon.svg
        assets/
      gallery/
        experience.json
        index.html
        icon.svg
        assets/
      notes/
        experience.json
        index.html
        icon.svg
        assets/
  schemas/
    catalog.schema.json
    experience.schema.json
    launcher.schema.json
  docs/
    architecture.md
    experience-integration.md
    theme-development.md
  tests/
    core/
    integration/
    themes/
  .github/
    workflows/
      deploy.yml
  dist/
```

`library` and `phone` illustrate the repeated theme structure; they do not select either theme for production or require both to be built initially. Other themes follow the same pattern. `dist/` is generated deployment output and should not be maintained as source.

### Folder responsibilities

| Folder | Responsibility |
| --- | --- |
| `src/core/` | Load and validate metadata, manage the active iframe and navigation state, validate messages and clean up sessions. No theme-specific DOM or imagery. `display.js` owns display state and browser fullscreen transitions. |
| `src/theme-system/` | Register trusted theme modules and select the active theme independently from catalogue entries. |
| `src/themes/<theme-id>/` | Theme-specific home, experience frame, navigation presentation, responsive styles and visual assets. |
| `src/shared/` | Accessible controls, structural styles and French launcher strings used across themes. |
| `src/pwa/` | Optional shared PWA setup and service-worker source; installation is independent of theme. |
| `public/` | Files copied to the published site while preserving their relative paths. |
| `public/apps/<app-id>/` | Self-contained built-in applications, each integrated through its own metadata and iframe. |
| `schemas/` | Validation contracts, shared by runtime validation and relevant development checks. |
| `docs/` | Launcher architecture, experience-team contract and theme-authoring instructions. |
| `tests/` | Behaviour checks for the shared core, application integration and essential theme interactions. |

### Theme boundaries

The core owns exactly one active iframe and its session. Themes render around that host through a common contract: render the home from catalogue data, expose an experience mount area, render navigation from shared state and dispose theme-owned resources. Themes trigger core actions such as launch and exit; they do not create their own iframe lifecycle or bypass message validation. Each theme supplies its own default experience frame, which may already use the full client area. Every theme declares its supported display modes. A theme that supports `page` or `fullscreen` must provide those states without a compulsory phone frame, decorative margins, fixed aspect ratio or maximum-width constraint. Resizing must preserve the existing iframe document and application state.

Keep an application's metadata independent of themes: teams do not create one manifest or build per launcher theme. Any built-in application is listed in the catalogue, not hidden inside a theme directory. Theme icons, backgrounds and decorative objects belong to the theme; application icons belong to the application.

Theme modules run as trusted launcher code, not as experience iframes. The registry imports only themes included in the launcher deployment. Selecting a theme by configuration must not execute arbitrary code from a metadata URL. Remove theme styles and listeners on disposal; scope theme styles to avoid affecting the core or other themes.

### Theme selection configuration

A proposed optional `public/launcher.json` keeps presentation settings separate from the application catalogue:

```json
{
  "theme": {
    "default": "library",
    "available": ["library", "phone"],
    "selection": "automatic",
    "allowManualSelection": true,
    "automatic": {
      "compact": "phone",
      "wide": "library"
    }
  }
}
```

This example demonstrates the configuration shape, not a chosen design. `selection` may be `fixed`, `automatic` or `manual`. In fixed mode use the default; in manual mode use a valid saved choice or the default; in automatic mode prefer a valid explicit choice when manual selection is enabled, otherwise use the viewport mapping and then the default. The registry validates every ID against the installed themes. A missing or invalid choice falls back to a valid default theme.

Define compact/wide using a documented viewport rule in `selection.js`, not browser-name detection. The breakpoint and reevaluation timing on the home screen remain open. Regardless of the eventual selection configuration, follow the confirmed [theme selection during play](specification.md#theme-selection-during-play) policy: apply theme selection only at home, keep the active theme while an experience runs and defer any requested switch until returning home. Viewport changes adapt the active theme's responsive layout while preserving the same iframe document and application state.

### Source paths and published paths

Configure the build to copy the contents of `public/` into the deployment root. The public URL never contains the source directory name `public`.

| Source | Published URL relative to the launcher base |
| --- | --- |
| `public/catalog.json` | `./catalog.json` |
| `public/launcher.json` | `./launcher.json` |
| `public/apps/notes/experience.json` | `./apps/notes/experience.json` |
| `public/apps/notes/index.html` | `./apps/notes/index.html` |

The existing catalogue examples therefore remain valid. Resolve runtime paths against the deployment base so GitHub project Pages and a dedicated domain both work. Bundle theme assets with their modules. If a PWA is implemented, emit its service worker at the correct public scope; its source location under `src/pwa/` is not its deployed URL.

## Authoritative contracts

See [API](api.md), [experience integration](experience-integration.md) and [theme development](theme-development.md).
