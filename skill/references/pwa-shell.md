# PWA shell reference

The installable app is the **Sveltia admin itself**. These are the things that make it
install + work offline on a GitHub *project* page. (Sveltia handles editing/encoding/sync —
this doc is only the shell.)

## The `/<repo>/` base-path gotcha (the #1 silent failure)

On a GitHub **project** page the site root is `https://<you>.github.io/<repo>/`, not `/`.
Absolute paths (`/manifest.json`, `/sw.js`) break. So everything is **relative**:

- `manifest.json`: `"start_url": "./admin/"`, `"scope": "./"`, `"id": "./"`. Resolved
  against the manifest's own URL (the site root) → `/<repo>/admin/` and `/<repo>/`.
- Register the SW from the **root** page with a relative path: `register('sw.js')`. Its
  scope becomes `/<repo>/`, which **covers `/admin/`**. (You can't widen a SW's scope from
  inside `/admin/` — that needs the `Service-Worker-Allowed` header, which Pages doesn't
  send. So the landing page is the registrar.)
- Icons/manifest links use relative hrefs (`../manifest.json` from `/admin/`).

> **User/org page** (`<you>.github.io`, repo named `<you>.github.io`) serves at root `/`,
> so the base-path problem disappears. The relative paths still work there — no change
> needed.

## Installability requirements (2026)

Chrome/Edge prompt on **valid manifest + HTTPS** (Pages gives HTTPS free), but still ship a
real **service worker with a `fetch` handler** for offline + to satisfy Android's
auto-prompt. The manifest must have: `name`, `short_name`, `start_url`, `display:
standalone`, `background_color`, `theme_color`, and icons including a **512×512** plus a
**192 maskable**.

Verify with Chrome DevTools → **Application → Manifest** (no errors, "Installable") and
**Lighthouse → PWA** audit.

## Offline strategy (see `sw.js`)

- **Precache, cache-first:** the app shell + the same-origin vendored `sveltia-cms.js`.
  This is the whole point of vendoring — a CDN URL can't be reliably offline-cached here.
- **Cross-origin untouched:** requests to `api.github.com` / avatars / media CDN are *not*
  intercepted — Sveltia's data layer hits the network directly; never cache writes.
- **Bump `CACHE`** in `sw.js` whenever the bundle is re-vendored, to evict the old file.

Offline = the editor loads and previously-loaded content shows; new sync resumes online.
