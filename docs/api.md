# Launcher API

Protocol specification before implementation. Product requirements, including the initial display mode, common non-fullscreen layout requirement, fullscreen failure fallback and restoration rules, are confirmed in [decisions.md](decisions.md). Envelope details and schema defaults remain proposed implementation choices. No API code is implemented in this repository.

## Optional immersive mode

Here, **immersive means all of the launcher's experience area**. It does not mean browser or operating-system fullscreen.

An experience may hide the launcher navigation only after its own French exit control is rendered, enabled and usable. That control must remain reachable throughout the immersive experience, including in menus and end screens. The iframe remains inside the launcher's shell. Navigation visibility is independent of display mode: hiding the bar does not request browser fullscreen, and changing display mode does not hide navigation.

The agreed commands are:

| Message type | Direction | Effect |
| --- | --- | --- |
| `launcher.navigation.hide` | Experience → launcher | Hide the navigation bar and expand the iframe |
| `launcher.navigation.show` | Experience → launcher | Restore the navigation bar |
| `experience.exit` | Experience → launcher | Remove the iframe and return home |
| `experience.ready` | Experience → launcher | Optional signal that application loading is complete; clears the launcher loading indicator |
| `experience.completed` | Experience → launcher | Optional completion signal; does not automatically exit |

Repeated hide/show requests should be harmless. A valid exit request uses the same immediate cleanup path as the shell's “Accueil” button: remove the iframe and return home without a confirmation dialog or save-before-close handshake. Do not wait for application saving before closing. See [leaving is not pausing](specification.md#leaving-is-not-pausing) for progress responsibilities. No permission prompt or metadata declaration is needed for these commands.

`experience.ready` is optional application readiness, not a mandatory protocol acknowledgement. It clears the loading indicator only when it passes normal active-session validation. Without it, the launcher dismisses the indicator after a short delay without asserting success or failure; see [loading feedback and failures](specification.md#loading-feedback-and-failures). Experiences without messaging still run. This does not resolve the separate handshake acknowledgement and retry design.

No additional launcher recovery control is provided when navigation is hidden. If an experience becomes stuck or its exit control fails, the player can refresh the launcher page or close and relaunch the installed PWA to return home. Restarting does not automatically reopen the experience or restore its in-memory state. The experience's own usable exit control remains required for normal operation.

### Flexible experience display API

No phone frame or other decorative container is mandatory on desktop. An experience or a theme control can request one of three modes, subject to negotiated support. Display mode remains independent of navigation visibility:

| Mode | Meaning |
| --- | --- |
| `theme` | Use the active theme's default experience layout. It may be a phone, window, panel, or already the full client area. |
| `page` | Remove decorative framing and theme-imposed size/aspect constraints. Expand the experience host to the launcher's entire client viewport, keeping any visible launcher navigation and required safe areas. Browser chrome remains. |
| `fullscreen` | Use an unframed, expanded layout and request native browser fullscreen for the launcher root containing both experience and controls. Subject to browser support, permissions and user activation. |

### Supported modes and launcher mediation

The experience declares `displayModes` in `experience.json`; each trusted theme declares the same field in its module descriptor, for example:

```js
export default {
  id: "library",
  displayModes: ["theme", "page", "fullscreen"]
  // Other lifecycle and rendering hooks follow the theme contract.
};
```

The descriptor field is mandatory for new themes. Arrays must contain recognised, unique mode values. Missing experience declarations default to `["theme"]` for compatibility, not to support for every mode. These display declarations do not introduce an immersive/navigation capability flag: navigation hide/show remains its own existing contract.

The core computes:

```text
commonModes = experience.displayModes ∩ activeTheme.displayModes
availableModes = commonModes filtered by current browser support and policy
```

Needing user activation does not remove `fullscreen` from available modes: it changes how the request is fulfilled. A known lack of Fullscreen API support or a policy restriction makes that control unavailable. Unexpected runtime rejection still requires a reported fallback.

The confirmed launch rule requires a common non-fullscreen layout (`theme` or `page`). An experience and theme may each support only a subset of modes, but an intersection containing only `fullscreen` is insufficient because users can always leave native fullscreen. Start in `theme` when it is common, otherwise `page`. If neither is common, block launch: keep the launcher home available, show French incompatibility feedback and offer a compatible installed theme if one exists. Do not silently switch themes during play.

Both callers use the core's display manager:

- Experience code sends `launcher.display.request` through the validated message channel.
- A trusted theme calls an injected launcher action such as `requestDisplayMode("page")`; it does not use `postMessage` to impersonate the experience.
- Both paths validate the same supported-mode intersection, browser constraints and current session, and perform the same state transition.
- Only the core updates effective mode, manages native fullscreen and publishes resulting state. The theme must not independently resize the host or call native fullscreen to bypass the manager.

The display manager exposes availability and state to the theme through the theme contract. Themes may hide unsupported controls or show them disabled with an accessible French explanation. Supported controls display the active mode and remain keyboard accessible. For instance, if the experience supports `["page", "fullscreen"]` and the theme supports `["theme", "page"]`, only `page` is common; the theme must hide or disable its framed-view and fullscreen buttons.

For a theme button that requests fullscreen, invoke the core action synchronously from its click handler. The core should call the browser API within that user activation where possible, avoiding the extra activation button required by an asynchronous message path.

### Availability notifications

Add `launcher.display.available` (launcher → experience), using the same channel, protocol version and session ID. Send it after the integration handshake and whenever relevant availability changes. Its payload contains the declarations and derived support:

```json
{
  "experienceModes": ["theme", "page", "fullscreen"],
  "themeModes": ["theme", "page", "fullscreen"],
  "commonModes": ["theme", "page", "fullscreen"],
  "availableModes": ["theme", "page"],
  "unavailableReasons": {
    "fullscreen": "unsupported"
  }
}
```

This JSON is the message payload, not the complete envelope. The theme receives the same derived data through an injected subscription, not by independently recomputing capabilities. Experiences may also adapt their own controls from this notification. No application data or capabilities from unrelated experiences should be included.

Requesting a mode not in `commonModes` returns `launcher.display.changed` with `status: "error"`, `reason: "mode-not-supported"` and the unchanged effective mode. This must not initiate a pending fullscreen request. A recognised but browser-unavailable fullscreen request uses the negotiated fallback behaviour below. Unknown mode strings remain `invalid-mode` errors.

To give the experience all available client space without the normal navigation bar, combine `page` with `launcher.navigation.hide`, after the experience's exit control is ready. Browser or operating-system UI and device safe areas cannot be promised away.

| Message type | Direction | Purpose |
| --- | --- | --- |
| `launcher.display.request` | Experience → launcher | Request `theme`, `page` or `fullscreen`. |
| `launcher.display.changed` | Launcher → experience | Report the effective mode and request outcome, including pending activation, fallback and native fullscreen exit. |
| `launcher.display.available` | Launcher → experience | Announce declared, common and currently available modes. |

Use the common channel, protocol version and session ID described below. Display requests also include a unique `requestId` and a `payload.mode`:

```json
{
  "channel": "from-play-to-prevention",
  "protocolVersion": 1,
  "sessionId": "launcher-generated-value",
  "type": "launcher.display.request",
  "requestId": "display-1",
  "payload": {
    "mode": "page"
  }
}
```

The launcher replies after applying the layout, using the same request ID:

```json
{
  "channel": "from-play-to-prevention",
  "protocolVersion": 1,
  "sessionId": "launcher-generated-value",
  "type": "launcher.display.changed",
  "requestId": "display-1",
  "payload": {
    "requestedMode": "page",
    "mode": "page",
    "status": "applied",
    "reason": null
  }
}
```

`mode` is always the effective mode, never merely a requested state. Publish changes from theme controls to the experience as well, with `requestId: null` and `source: "theme"` in the payload; use `source: "experience"` for experience requests, `"browser"` for native external transitions and `"launcher"` for other core actions. The theme receives every state update through its subscription. Supported outcome values are:

| `status` | Meaning and expected state |
| --- | --- |
| `applied` | Requested state has been applied. |
| `pending` | Fullscreen needs explicit user activation; effective mode remains a common non-fullscreen layout. Show a launcher-owned French button such as “Activer le plein écran”. |
| `fallback` | Fullscreen is unavailable, rejected, cancelled or exited externally; report the effective non-fullscreen mode. |
| `error` | Invalid mode or request; preserve the current effective state. |

`reason` is `null` for success, otherwise a machine-readable English code such as `user-activation-required`, `unsupported`, `denied`, `cancelled`, `superseded`, `browser-exit` or `invalid-mode`. User-facing explanations are French. A pending request must eventually receive an outcome; cancelling the activation control reports `fallback` with `cancelled` while keeping the negotiated non-fullscreen fallback. A newer request resolves any older pending request as `fallback` with `superseded` before being processed. Cancel pending controls when the session ends; do not send messages to an exited application.

For changes initiated outside the experience request flow, such as the browser's fullscreen exit control, send a state notification with `requestId: null`. `requestedMode` records the last requested mode; `mode` and `status` explain what actually happened. Do not use this state event to report navigation visibility, which remains a separate concern.

### Fullscreen transitions and restoration

The initial mode, fullscreen entry-failure fallback, restoration on browser exit and iframe preservation rules below are confirmed. Pending-request arbitration and pre-existing launcher fullscreen ownership remain [open questions](open-questions.md#display-api).

- Start each experience with navigation visible in its negotiated initial mode: common `theme`, otherwise common `page`. Support declarations constrain both theme and experience requests.
- Validate source window, origin, session, request ID, message shape and allowed mode using the same safeguards as other API messages.
- Preserve the running iframe throughout all display transitions, including fullscreen entry, failure and exit: do not reload, recreate or navigate it. Notify the experience after state changes; applications must handle normal viewport resizing.
- Treat an incoming `postMessage` as a request, not a guaranteed browser user gesture. Native fullscreen requires transient user activation. If the parent lacks it, show the launcher-owned activation button and invoke `requestFullscreen()` directly inside its click handler, without awaiting unrelated asynchronous work first.
- Keep that activation button reachable even when normal launcher navigation is hidden. It can be a temporary control; dismissing it leaves the experience in its negotiated non-fullscreen fallback.
- Check browser support and policy, handle the request promise rejection, and use `fullscreenchange` plus `document.fullscreenElement` to track actual fullscreen state. Never announce `fullscreen` before native entry succeeds.
- Before requesting native fullscreen, remember the prior common non-fullscreen layout (`theme` or `page`). If the browser exits fullscreen, restore that layout and notify the active experience with `fallback` and `browser-exit`. If fullscreen was unavailable or denied on entry, use common `page` when available, otherwise common `theme`, and report that fallback. Never choose a mode outside the common set.
- An explicit request for a supported `theme` or `page` while fullscreen first exits launcher-owned native fullscreen, then applies the requested layout. If exiting fails, report `error` and the actual effective state.
- On experience exit, cancel pending activation, restore navigation and reset display state. If fullscreen was entered for that experience, exit it; preserve any fullscreen session the user had independently established for the launcher before the experience. Track fullscreen ownership separately from the requested application mode.
- Do not intercept the browser's native exit behaviour or automatically force fullscreen again after the user leaves it.

The launcher owns the native fullscreen element so that launcher controls remain within the fullscreen root; Home is shown when navigation is visible. Fullscreen does not change navigation visibility or add a recovery control. Experience teams request the mode through this API instead of placing only their iframe contents into fullscreen. The same contract applies on mobile where supported, on desktop, across themes and to built-in applications.

## Recommended message protocol and examples

Use `window.postMessage()` for both local and external applications. The following namespaced envelope, handshake and session token are implementation recommendations to make the agreed commands reliable; they were not additional product features.

### Message envelope

```json
{
  "channel": "from-play-to-prevention",
  "protocolVersion": 1,
  "sessionId": "launcher-generated-value",
  "type": "experience.exit"
}
```

The launcher creates a new unpredictable session ID for each iframe load. It sends a `launcher.init` message after the iframe load event, addressed to the entry URL's exact origin. The experience must install its listener early during startup. Applications that delay listener registration need an agreed retry or acknowledgement mechanism; do not assume late listeners receive old messages.

### Minimal experience-side integration

```js
const CHANNEL = "from-play-to-prevention";

// Populate this with the actual launcher deployment origins.
// An origin has no path: https://organisation.github.io is an origin.
const ALLOWED_LAUNCHER_ORIGINS = new Set([
  "https://launcher.example.org"
]);

let launcherSession = null;
let exitControlReady = false;

function sendToLauncher(type, payload = undefined, requestId = undefined) {
  if (!launcherSession) return;
  window.parent.postMessage({
    channel: CHANNEL,
    protocolVersion: 1,
    sessionId: launcherSession.id,
    type,
    ...(payload ? { payload } : {}),
    ...(requestId ? { requestId } : {})
  }, launcherSession.origin);
}

window.addEventListener("message", (event) => {
  if (window.parent === window || event.source !== window.parent) return;
  if (!ALLOWED_LAUNCHER_ORIGINS.has(event.origin)) return;

  const message = event.data;
  if (!message || typeof message !== "object") return;
  if (message.channel !== CHANNEL || message.protocolVersion !== 1) return;
  if (message.type !== "launcher.init") return;
  if (typeof message.sessionId !== "string" || !message.sessionId) return;

  launcherSession = { id: message.sessionId, origin: event.origin };
  sendToLauncher("experience.ready");
  if (exitControlReady) sendToLauncher("launcher.navigation.hide");
});

// Call only once the application's exit button is rendered and usable.
function enableImmersiveMode() {
  exitControlReady = true;
  sendToLauncher("launcher.navigation.hide");
}

// Optional: expand beyond the theme frame without hiding navigation.
function expandExperience() {
  sendToLauncher("launcher.display.request", { mode: "page" }, crypto.randomUUID());
}

function leaveExperience() {
  if (launcherSession) {
    sendToLauncher("experience.exit");
  } else {
    // This example assumes a standalone menu with this ID exists.
    document.getElementById("standalone-menu")?.focus();
  }
}
```

This is an integration excerpt, not a complete app. Bind `leaveExperience()` to an actual button labelled, for example, “Retour à l’accueil”. A non-immersive application does not call `enableImmersiveMode()`. An application with no messaging code still runs with launcher navigation visible.

For standalone use, provide an appropriate own menu or restart screen. Do not silently call a nonexistent parent integration. Do not trust an arbitrary query parameter as the allowed parent origin.

### Launcher-side validation

Before accepting any experience message, require all of the following:

1. `event.source === activeIframe.contentWindow`.
2. `event.origin` equals the active entry URL's expected origin.
3. Channel and protocol version match.
4. Session ID matches the current iframe load.
5. Message type is a recognised command and its payload is valid.

Use an exact target origin when sending messages; do not use `"*"`. Do not redirect the entry page to another origin without updating the integration configuration. Discard messages after exit. Restore focus to an appropriate exit control when hiding a bar that currently contains keyboard focus.
