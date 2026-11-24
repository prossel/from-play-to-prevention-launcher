# Experience integration guide

For independent teams and built-in applications. Public interface text is French; retain the supplied experience titles. All URLs in examples are placeholders, not registered deployments.

## Catalogue and metadata contract

The following schema is the proposed version 1 contract. A framework or backend is not required.

### Launcher catalogue

```json
{
  "apps": [
    "./apps/camera/experience.json",
    "./apps/gallery/experience.json",
    "./apps/notes/experience.json",
    "https://team-a.github.io/parallel-group-chat/experience.json"
  ]
}
```

Array order determines the initial icon order. Read each URL relative to the fetched catalogue's URL. Adding a project requires adding its metadata URL and publishing the catalogue; updating an existing application does not require rebuilding the launcher.

### Application metadata

```json
{
  "schemaVersion": 1,
  "id": "parallel-group-chat",
  "title": "Le groupe parallèle",
  "description": "Une histoire de groupe, de captures et de choix.",
  "icon": "./icon.png",
  "entry": "./index.html",
  "language": "fr",
  "version": "1.0.0",
  "displayModes": ["theme", "page", "fullscreen"]
}
```

| Field | Requirement | Meaning |
| --- | --- | --- |
| `schemaVersion` | Required | Metadata schema version; currently `1` |
| `id` | Required | Stable English identifier, unique across the catalogue |
| `title` | Required | French display title |
| `description` | Optional | Short French summary |
| `icon` | Required | Relative or absolute URL to a square icon; SVG or PNG recommended |
| `entry` | Required | Relative or absolute URL to the published application page |
| `language` | Optional | Content language; normally `fr` |
| `version` | Optional | Team-managed release label; not a cache-busting mechanism by itself |
| `displayModes` | Optional for legacy manifests | Supported values from `theme`, `page`, `fullscreen`. Omitted means `["theme"]`; new experiences should declare their modes explicitly. |

Do not add `capabilities.immersive` or an equivalent flag: the runtime request to hide navigation is sufficient.

Resolve `icon` and `entry` against the metadata document's URL, preferably its final response URL after redirects. Use URL resolution, not string concatenation. Avoid leading `/` paths for repository-relative assets: GitHub project sites commonly live below a repository path, while a custom domain may have a different base path.

Only catalogue-approved applications should load. Validate metadata, reject duplicate IDs and unsupported schema versions, allow HTTPS deployment URLs, and render metadata text as text rather than HTML. Local development may use localhost HTTP.

### Failure behaviour

Fetch catalogue entries independently. One invalid or unreachable manifest must not prevent other applications from appearing. Show French feedback such as “Cette expérience est indisponible pour le moment.” with a retry action. Use a fallback icon when only the icon is missing. Unsupported metadata must be reported clearly to maintainers.

## Hosting and browser boundaries

### Embedding and metadata access are separate

The experience page must permit iframe embedding: `Content-Security-Policy: frame-ancestors` and `X-Frame-Options`, when present, must be compatible with the launcher. The launcher must also permit the experience's source in its own framing policy.

The metadata JSON must additionally be readable with browser `fetch()`. If hosted on another origin, its server must provide appropriate CORS headers. CORS is not what authorises rendering an iframe. Validate both behaviours against the actual published URLs; do not try to fix server policies with an HTML meta tag or `fetch(..., { mode: "no-cors" })`.

### Iframes are not automatically a security boundary

Choose sandbox and feature permissions deliberately, and test them against participating technologies. Different GitHub Pages repository paths on the same hostname share an origin. Built-in apps also share the launcher origin. Without effective isolation, same-origin applications may access parent DOM and share origin storage; treat them as trusted code. A sandbox combining scripts and same-origin access does not provide strong isolation from trusted same-origin scripts.

For stronger separation of independently maintained code, use separate deployment origins and a tested sandbox policy. Restrict top-level navigation and unsolicited popups where compatible. Do not let an experience replace the launcher page to implement exit.

Camera or microphone access, if actually required, needs explicit feature-permission configuration, browser permission and device testing. A simulated camera avoids that dependency. Determine these requirements per application rather than granting every iframe every feature.

## Deployment and automatic updates

### Independent experience deployment

1. Each team chooses its stack and maintains its own repository.
2. A push to `prod` triggers its GitHub Actions deployment workflow.
3. The workflow builds if needed and publishes the static output to GitHub Pages.
4. The published output includes `experience.json`, its icon and the entry page.
5. The stable metadata URL is registered once in the launcher catalogue.
6. Later successful deployments become available at those same URLs without changing launcher code.

GitHub Pages can also publish directly from a selected branch for simple static projects. The repository must be configured for the chosen publishing method. A repository URL alone is insufficient: the launcher needs an actual published metadata endpoint. Build-time projects must configure their asset base paths for their deployment URL.

The launcher has its own deployment workflow. Its built-in applications are updated when that repository deploys. No shared build process across all team repositories is required.

### Refresh and caching policy

Revalidate catalogue metadata when opening the launcher and returning home. Apply catalogue changes on the home screen, never by replacing a running experience. On the next launch, load the current entry URL. Browser/CDN caches and application service workers mean that “push” does not imply instantaneous delivery to every open device.

For the first version, avoid caching external experience documents in the launcher's service worker. Prefer revalidated entry documents and content-hashed build assets where supported. A metadata `version` field is informational unless an explicit versioned-URL strategy is implemented. Preserve a cached catalogue only as an optional fallback and show unavailable applications honestly.

## Delivery checklist for experience teams

- Publish a working HTTPS application with French user-facing content.
- Supply valid version 1 metadata, a square icon and a stable entry URL.
- Make assets work under the actual GitHub Pages repository path.
- Declare supported `displayModes` and handle resizing within those modes. Support the available experience viewport with navigation visible and hidden; avoid fixed device-pixel dimensions.
- Test touch, keyboard, focus, readable text and scrolling.
- Permit iframe embedding and cross-origin metadata fetching where needed.
- Keep the shell visible unless an implemented exit control can replace it.
- If messaging is used, implement exact-origin validation and the agreed envelope.
- Support standalone loading with sensible controls and no dependency on a parent frame.
- Document progress persistence and any real hardware permissions.
- Test the published version inside the launcher, not only the local development page.

## Runtime integration

The authoritative message protocol, display negotiation and navigation commands are in [api.md](api.md). Experiences advertise displayModes; the launcher mediates both theme and experience requests. No immersive metadata flag is needed.

## Current projects

- Tik Tak Tik Tak
- The Secret of St Christophe
- Marked
- Without a Trace
- Echo
- Le groupe parallèle

Repository URLs, owners, deployment URLs, icons and content summaries remain to be supplied. Do not infer storylines from project names.
