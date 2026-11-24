# Experience integration guide

For independent teams and built-in applications. Public interface text is French; retain the supplied experience titles. All URLs in examples are placeholders, not registered deployments.

## Start here: minimum integration

Develop, run and test the experience independently in its own repository, with its own tools. It must remain usable when opened directly, without waiting for a launcher handshake or importing launcher code.

For minimum integration, publish a working HTTPS experience and register only a stable ID and its published folder or page URL in the launcher catalogue. The launcher discovers the display name and icon from the page, with optional `experience.json` overrides. No launcher-specific JavaScript, SDK, build dependency, icon or `experience.json` is required in the experience repository.

The published URL must open the experience directly and allow iframe embedding. A folder URL ending in `/` normally serves `index.html` through the hosting server; verify this in a browser. The launcher opens that URL as supplied, rather than guessing a filename. If the host does not serve a folder index, supply the actual page URL instead. A Git repository URL is not a published experience URL.

The experience must fit a responsive iframe, work with touch and keyboard, and keep its assets working under its published path. These are compatibility requirements even when no integration code is needed. At this level the launcher attempts page-title and favicon discovery, keeps navigation visible and assumes `displayModes: ["theme"]`. The usual shared-mode compatibility check still applies.

## Team workflow: fork, test, submit

The launcher and the test workflow described here are specified, not implemented yet. Commands and actual launcher origins will be documented once the implementation and hosting are selected.

1. Develop the experience standalone and publish a test version at a stable URL.
2. Fork the launcher repository and add an entry to the fork's catalogue using that URL. Supply a unique stable English ID; this registration work does not require changing experience code.
3. Run the fork locally or publish it as a test launcher. An independently published experience does not require a shared build with the launcher.
4. Test opening, loading feedback, resizing, touch, keyboard, “Accueil”, audio stopping on exit and reopening. Test any optional integration features you use, across compatible themes and declared display modes.
5. Sync the fork with the upstream launcher and test again. If using messaging or a hosting allowlist, configure the actual local, fork and production launcher origins explicitly.
6. Replace any temporary URL with the stable production URL, verify it in the fork, then submit a pull request to the upstream launcher containing the catalogue addition. Keep unrelated launcher changes out of that PR. No experience source code is copied into the launcher repository.
7. After review, merge and launcher publication, verify the experience in the actual production launcher, including its origin and embedding policies. Later experience updates at the same URL use the team's own deployment process.

Local experience development alongside a local launcher is optional for faster integration iteration. It is never required for ordinary experience development. Localhost HTTP is allowed for local testing; published entries use HTTPS. Test on a genuinely different origin before acceptance.

## Catalogue and metadata contract

Folder-URL registration and optional metadata replace the earlier mandatory metadata-endpoint design. The following JSON field names and discovery convention are proposed version 1 schema details; no runtime or schema files are implemented.

### Launcher catalogue

```json
{
  "apps": [
    {
      "id": "parallel-group-chat",
      "url": "https://team-a.github.io/parallel-group-chat/"
    }
  ]
}
```

| Field | Requirement | Meaning |
| --- | --- | --- |
| `id` | Required in catalogue | Stable English identifier, unique across the catalogue |
| `url` | Required in catalogue | Published folder URL ending in `/`, or explicit entry page URL |

Array order sets initial icon order. Resolve relative URLs against the fetched catalogue URL. A built-in entry can use a URL such as `./apps/notes/` with the same contract. The catalogue contains only `id` and `url`; names, icons and descriptions belong to the experience page or optional metadata. Do not infer titles or content from IDs. Only approved catalogue entries may load. Validate entries, reject duplicate IDs and invalid deployment URLs, and render text as text rather than HTML.

### Automatic name and icon discovery

Resolve the display name in this order: non-empty optional metadata `title`, non-empty HTML `<title>`, then the catalogue `id` as a temporary fallback label. Use the resolved name for both the home entry and the accessible iframe title. Preserve supplied titles exactly; do not translate or turn an ID into an invented title. Reviewers must check that the page title or override matches the intended experience name.

Resolve the launcher icon in this order: usable optional metadata `icon`, usable HTML `<link rel="icon">` candidate, the experience origin's `/favicon.ico`, then the launcher's generic fallback icon. Try the next source if an image cannot load. The root favicon is host-wide, not relative to the experience folder; repository sites sharing a hostname may need a page icon link or JSON override for a distinct icon. Candidate selection details for multiple sizes, types and media remain proposed implementation work. See [MDN's icon-link documentation](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Attributes/rel#icon).

Read the published HTML as data without executing its scripts or mounting hidden experiences. Extract only title text and icon links; do not insert fetched markup into the launcher. Resolve relative icon links against the final page response URL, accounting for a valid HTML base URL, and validate resulting URLs before use. Page scripts that change the title or icon after startup are not part of static discovery. Supply static head tags or optional JSON for reliable presentation.

Cross-origin HTML reading needs CORS, even though merely embedding the same page does not. A readable `experience.json` can provide a name and icon when HTML is unreadable; if neither is readable, the ID remains the label and a root favicon may still be displayed. The launcher cannot read a cross-origin iframe's document to bypass this restriction. No proxy service is required for baseline integration. See [MDN's same-origin policy](https://developer.mozilla.org/en-US/docs/Web/Security/Defenses/Same-origin_policy).

Discovery is best effort and must not block baseline launching indefinitely. Names and icons can refresh on the home screen; they do not require executing or restarting the experience. Display declarations continue to follow the session rules below.

### Optional level 1: metadata without messaging

Publish `experience.json` alongside the entry page to override its display title or icon, or supply a description or explicit display support. Proposed discovery resolves `./experience.json` against the catalogue launch URL, so both a folder URL and its `index.html` page resolve to the same metadata location. Register the final stable launch URL rather than relying on cross-origin redirects.

```json
{
  "schemaVersion": 1,
  "title": "Le groupe parallèle",
  "description": "Description française fournie par l’équipe.",
  "icon": "./icon.png",
  "language": "fr",
  "version": "1.0.0",
  "displayModes": ["theme", "page", "fullscreen"]
}
```

The description above is a placeholder, not a supplied summary of any named experience.

| Field | Requirement | Meaning |
| --- | --- | --- |
| `schemaVersion` | Required when metadata is supplied | Metadata schema version; currently `1` |
| `title` | Optional | Display-name override; preserve the supplied experience title exactly, including English titles |
| `description` | Optional | Short French summary supplied by the team |
| `icon` | Optional | Relative or absolute URL to a square icon; SVG or PNG recommended |
| `language` | Optional | Content language; defaults to `fr` |
| `version` | Optional | Team-managed release label; does not itself invalidate caches |
| `displayModes` | Optional | Recognised unique modes from `theme`, `page`, `fullscreen`; omitted defaults to `["theme"]` |

Stable identity and launch URL stay in the catalogue; metadata `title` overrides the HTML page title. Resolve the icon URL against the metadata document's final response URL using URL resolution, not string concatenation. Avoid leading `/` for repository-relative assets. Explicit display declarations must match tested support; metadata alone does not require messaging. See [api.md](api.md) for display negotiation.

Missing metadata, CORS failure, invalid JSON or an unsupported metadata version must not block the baseline catalogue entry. Retain available page-derived presentation and baseline display defaults, and report unusable optional metadata to maintainers. If an override icon fails, continue through the icon fallback chain below. Optional metadata discovery must not hold the catalogue or baseline launch indefinitely; the exact bounded fetch policy remains open. Resolve declarations before launch and keep them fixed for that running session; late metadata applies only to a later launch.

### Optional level 2: launcher messages

Messaging can be added without metadata when the default `theme` display support is sufficient. The optional levels are independent additions, not mandatory stages. Add only the messages your experience needs, using the exact-origin and session validation in [api.md](api.md):

| Feature | Additional work |
| --- | --- |
| Clear launcher loading feedback when ready | Send optional `experience.ready` |
| Return to launcher home | Send `experience.exit`; retain a sensible standalone menu when no launcher connection exists |
| Hide/show launcher navigation | Provide a working French exit control before hiding it, then send the navigation commands |
| Request a display mode or adapt controls to availability | Use the display API; declare additional tested modes in optional metadata when needed |
| Report completion | Send optional `experience.completed`; it does not automatically close the experience |

No `capabilities.immersive` flag is required. Do not wait for a launcher connection before allowing standalone play. Without messaging, launcher navigation provides exit. With navigation hidden, the experience's own exit must remain usable; stuck sessions use the [restart policy](specification.md#recovery-from-a-stuck-experience).

## Hosting, embedding and CORS

### What requires CORS?

| Operation | Requirement |
| --- | --- |
| Open the experience URL in an iframe | No CORS header is required merely for embedding; hosting policies must permit framing |
| Launcher reads another origin’s HTML to discover title and icon links | HTML response needs an appropriate `Access-Control-Allow-Origin` header |
| Launcher fetches optional `experience.json` from another origin | JSON response needs an appropriate `Access-Control-Allow-Origin` header |
| Launcher displays an icon using a normal image element | Normally no CORS header needed just to display it; image hosting and launcher policies must permit loading |
| Experience exchanges `postMessage` messages with launcher | No CORS header required; validate sender window, exact origin and session |
| Experience fetches its own APIs or third-party assets | Follow those services' access rules independently of launcher integration |

An origin includes scheme, hostname and port, without a path. Different repository paths under the same GitHub Pages hostname share an origin; different localhost ports do not. See [MDN's same-origin policy](https://developer.mozilla.org/en-US/docs/Web/Security/Defenses/Same-origin_policy).

### Page discovery and optional metadata CORS configuration

**GitHub Pages normally needs no CORS configuration.** It normally serves public files with `Access-Control-Allow-Origin: *`, allowing credential-free HTML and optional JSON discovery. Teams cannot configure custom response headers directly on GitHub Pages; do not add a `_headers` file or an HTML meta tag expecting it to change CORS. See [GitHub's response on custom header support](https://github.com/orgs/community/discussions/54257#discussioncomment-5782562).

Verify access on the actual published experience URL and, if supplied, its `experience.json` URL. Test discovery from the local or fork launcher, inspect those responses in the browser Network panel and check for CORS errors in the console. A page opening directly does not prove that another origin can read it. Custom domains, redirects or proxies may affect the final responses; verify those URLs too. This is a verification step, not a requirement for teams to configure headers when access already works.

The header-configuration guidance below applies to hosts or proxies that support custom response headers, when their defaults do not already allow access.

Fetch page HTML and optional JSON without credentials (`credentials: "omit"`). To allow cross-origin discovery, the corresponding response can use:

```http
Access-Control-Allow-Origin: *
```

Alternatively, allow one exact launcher origin. To support multiple approved origins, return the matching origin after checking the request's `Origin` against an allowlist, with `Vary: Origin`; do not put a comma-separated origin list in this header. Include local and fork origins when needed.

Where supported, configure HTTP response headers at the host. `mode: "no-cors"` does not make HTML or JSON readable, and an HTML meta tag cannot supply CORS headers. If HTML or metadata is unreadable, baseline launching still works using the remaining discovery sources and fallbacks. See [MDN's CORS guide](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS).

### Iframe embedding policy

The experience response's `Content-Security-Policy: frame-ancestors` and `X-Frame-Options`, when present, must permit the actual launcher. `X-Frame-Options: DENY` blocks embedding; `SAMEORIGIN` blocks a launcher on another origin. An `Access-Control-Allow-Origin` header does not override framing restrictions. The launcher must also allow the experience source in its own framing policy. Configure these policies as response headers, not HTML meta tags. See [MDN's X-Frame-Options reference](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/X-Frame-Options) and [frame-ancestors reference](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Content-Security-Policy/frame-ancestors).

If a published folder URL opens directly but fails in the launcher, inspect framing policies first. If the iframe works but its name, icon, description or declared modes are missing, inspect the HTML and optional metadata requests, CORS and validation diagnostics. Neither the iframe load event nor CORS access proves successful rendering.

### Iframes are not automatically a security boundary

Choose sandbox and feature permissions deliberately, and test them against participating technologies. Different GitHub Pages repository paths on the same hostname share an origin. Built-in apps also share the launcher origin. Without effective isolation, same-origin applications may access parent DOM and share origin storage; treat them as trusted code. A sandbox combining scripts and same-origin access does not provide strong isolation from trusted same-origin scripts.

For stronger separation of independently maintained code, use separate deployment origins and a tested sandbox policy. Restrict top-level navigation and unsolicited popups where compatible. Do not let an experience replace the launcher page to implement exit.

Camera or microphone access, if actually required, needs explicit feature-permission configuration, browser permission and device testing. A simulated camera avoids that dependency. Determine these requirements per application rather than granting every iframe every feature.

## Deployment and automatic updates

### Independent experience deployment

1. Each team chooses its stack and maintains its own repository.
2. A push to the team's chosen publication branch (for example, `prod`) can trigger its GitHub Actions deployment workflow; the branch and publishing method remain to be selected.
3. The workflow builds if needed and publishes the static output to GitHub Pages.
4. The published output includes the entry page and its assets; `experience.json` and a custom launcher icon are optional.
5. The stable published folder or entry page URL is registered once in the launcher catalogue.
6. Later successful deployments become available at those same URLs without changing launcher code.

GitHub Pages can also publish directly from a selected branch for simple static projects. The repository must be configured for the chosen publishing method. A repository URL alone is insufficient: the launcher needs the published experience URL. Build-time projects must configure their asset base paths for their deployment URL.

The launcher has its own deployment workflow. Its built-in applications are updated when that repository deploys. No shared build process across all team repositories is required.

### Refresh and caching policy

Revalidate the catalogue, page-derived presentation and any available optional metadata when opening the launcher and returning home. Apply catalogue changes on the home screen, never by replacing a running experience. On the next launch, load the current entry URL. Browser/CDN caches and application service workers mean that “push” does not imply instantaneous delivery to every open device.

For the first version, avoid caching external experience documents in the launcher's service worker. Prefer revalidated entry documents and content-hashed build assets where supported. A metadata `version` field is informational unless an explicit versioned-URL strategy is implemented. Preserve a cached catalogue only as an optional fallback and show unavailable applications honestly.

## Delivery checklist for experience teams

### Minimum

- Publish a standalone HTTPS experience with French interface content and the supplied title unchanged.
- Register only its stable ID and published folder or page URL in the fork's catalogue.
- Check that the resolved name and icon are correct: page title/favicon first, optional JSON overrides when needed. Verify cross-origin HTML access on the published URL; GitHub Pages normally needs no CORS configuration. Missing discovery does not block launch.
- Verify folder-index behaviour, asset paths, iframe embedding, responsive layout, touch and keyboard.
- Test the published experience through the fork; submit the catalogue PR when ready.
- Document any progress persistence and hardware needs. Save during play if persistence is needed: [exit is immediate](specification.md#leaving-is-not-pausing).

### Only when using optional features

- If publishing metadata, provide valid version 1 JSON and verify cross-origin access to its published URL; GitHub Pages normally needs no CORS configuration. Use `title` and `icon` only when overriding page discovery.
- If supplying an icon, provide a working square image URL.
- If declaring additional display modes, test them and handle resizing with navigation visible and hidden.
- If using messages, validate exact origins and the session envelope; retain standalone behaviour.
- If hiding navigation, provide a usable French exit control first.
- If using real hardware, document and test the required permissions with the launcher maintainers.

## Runtime integration

The authoritative message protocol, display negotiation and navigation commands are in [api.md](api.md). Optional metadata advertises `displayModes` (otherwise `["theme"]`); the launcher mediates both theme and experience requests. No immersive metadata flag is needed.

## Current projects

- Tik Tak Tik Tak
- The Secret of St Christophe
- Marked
- Without a Trace
- Echo
- Le groupe parallèle

Repository URLs, owners, deployment URLs, icons and content summaries remain to be supplied. Do not infer storylines from project names.
