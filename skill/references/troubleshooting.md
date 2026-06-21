# Troubleshooting

## App won't install / no install prompt
- **Manifest errors** → DevTools → Application → Manifest. Must be valid JSON, reachable at
  `/<repo>/manifest.json`, with 192 + 512 icons.
- **No service worker** → it must register and control the page. Registered from the
  **root** landing page (`register('sw.js')`), scope `/<repo>/`. Check Application → Service
  Workers shows it *activated*.
- **Base-path mismatch** → using absolute `/sw.js` or `/manifest.json` on a project page
  breaks silently. Must be relative. See `pwa-shell.md`.
- **Not HTTPS** → Pages is HTTPS by default; custom domains need HTTPS enforced.

## `/admin/` is blank
- The `<script src>` must point at the same-origin `./sveltia-cms.js` and the file must
  exist (it's ~1.9 MB). A 404 there = blank screen. Re-vendor per the template's `VENDOR.md`.
- Open the console — a YAML parse error in `config.yml` halts Sveltia. Validate indentation.

## Token login fails
- 404 on a private repo → token can't see it (see `token-scopes.md`).
- 403 → missing write permission.
- Wrong `backend.repo` → it points at a repo the token/owner doesn't have. Confirm
  `owner/repo` exactly.

## Edits don't persist / wrong repo
- A note saved but not where expected → `backend.repo` points at the *shell* repo instead of
  the *notes* repo (private mode). Fix `config.yml`.
- 409 / conflict → editing the same file from two places; reload to pull latest (Sveltia
  handles SHAs, just refresh).

## Privacy check fails (private mode)
- `…github.io/<repo>/knowledge/note/welcome.md` returns **200** instead of 404 → you put
  notes in the *public shell* repo. In private mode the shell repo must contain **no**
  notes; notes live only in the private repo. Move them out and let `backend.repo` point at
  the private repo.

## Pages not live
- First deploy lags ~1 min. Check repo → Settings → Pages for the published URL and build
  status, or `gh api repos/<you>/<repo>/pages`.
- Build from branch `main`, folder `/ (root)`.

## Accents / emoji look mangled
- Sveltia writes UTF-8; if a file looks wrong, it was likely committed by another tool with
  bad encoding. Re-save through the editor.

## Offline shows a login screen (expected)
- Cold-starting the installed app offline loads the shell but lands on Sveltia's login
  screen — **this is expected**, not a bug. GitHub is the backend; Sveltia re-checks auth
  against GitHub on startup, which needs network. Offline-authenticated editing/browsing is a
  Sveltia design boundary (see [#630](https://github.com/sveltia/sveltia-cms/issues/630)).
- If instead you get *"configuration file could not be retrieved"* offline, the service
  worker is stale (pre-v2). Reload online once (or clear the site's storage / reinstall the
  PWA) so the current SW — which precaches `admin/config.yml` — takes over.
- Installed PWA stuck on an old SW? It self-updates from `/admin/` now, but a one-time
  break-out (reinstall or **Settings → Apps → FockNote → Storage → Clear storage**) is needed
  to leave a pre-update version. (The "version" shown in Android app info is the WebAPK
  build number, unrelated to the FockNote service-worker cache version.)
