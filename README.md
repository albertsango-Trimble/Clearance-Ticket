# Trimble Connect — Top-Down Viewer Extension

A **Project Extension** panel that lives in the Explorer sidebar. Select one
or more design files, click **View top-down**, and it opens an embedded 3D
Viewer showing all selected files overlaid, camera locked to a plan
(top-down) view.

## Files

- `manifest.json` — the extension manifest Trimble Connect reads to install the panel.
- `panel.html` — the whole extension (HTML/CSS/JS in one file, no build step).

## How it works

1. The panel connects to the host Explorer with `WorkspaceAPI.connect(window.parent, ...)`.
2. It listens for the `extension.fileSelected` event to track which files are
   currently selected, and lets the user tick/untick them.
3. On **View top-down**, it:
   - requests an access token from the host (`extension.requestPermission("accesstoken")`),
   - embeds the Trimble Connect 3D Viewer in an `<iframe>` and opens a
     *second* Workspace API connection to it,
   - hands it the token (`embed.setTokens`) and initializes it for the
     current project (`embed.init3DViewer`),
   - loads every selected file as a model (`viewer.toggleModelVersion`, which
     accepts an array — the viewer natively supports overlaying multiple
     models),
   - sets the camera to the built-in `"top"` preset (`viewer.setCamera`).

## Installing it in a project

1. Host `manifest.json` and `panel.html` somewhere with HTTPS (any static
   host / CDN works). Update the `icon` and `url` fields in `manifest.json`
   to point at your hosted `panel.html`.
2. In Trimble Connect, open the project → **Settings → Extensions** →
   add a custom extension by manifest URL, pointing at your hosted
   `manifest.json`.
3. Enable it. It will appear as a panel extension in the project.

## Things worth double-checking before you ship this

The public TypeDoc reference for `trimble-connect-workspace-api` doesn't
publish exact payload shapes for a couple of items, so I've written the code
defensively and flagged it inline:

- **`extension.fileSelected` payload shape.** The docs confirm the event
  name and that it supports multi-select, but not whether `args.data` is a
  single `ConnectFile`, `{ file, source }`, or `{ files: ConnectFile[] }`.
  `panel.html` handles all three shapes — worth confirming which one you
  actually get by `console.log`-ing `args` once installed.
- **`embed.init3DViewer` timing.** The guide confirms the method and that it
  accepts `{ projectId, modelId, versionId, viewId, ... }`, but doesn't
  document a "ready" event to await before loading additional models. The
  code calls `toggleModelVersion` right after `init3DViewer` resolves; if
  models don't appear reliably, add a short delay or listen for a
  `viewer.onModelStateChanged` event before loading.
- **Region.** `https://3d.connect.trimble.com` is the default (US/global)
  viewer host. If your project lives in the EU region, use the matching
  region viewer URL instead.

## References

- Workspace API overview & extension types: https://components.connect.trimble.com/trimble-connect-workspace-api/index.html
- Full API reference (all interfaces/events): https://components.connect.trimble.com/trimble-connect-workspace-api/modules.html
- `ViewerAPI` (setCamera, toggleModelVersion, etc.): https://components.connect.trimble.com/trimble-connect-workspace-api/interfaces/ViewerAPI.html
- Embedding a component (tokens, init3DViewer): https://developer.trimble.com/docs/connect/guides/embed/
