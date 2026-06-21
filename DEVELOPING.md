# Developing & testing FockNote

Three ways to work with this repo:

1. [Local development & testing](#1-local-development--testing) — edit the shell, see it in a browser.
2. [Running the skill in chat (Tier B)](#2-running-the-skill-in-chat-tier-b) — guided, no write access.
3. [Running the skill in Claude Code (Tier A)](#3-running-the-skill-in-claude-code-tier-a) — fully automated.

The app is **build-step-free**: plain HTML/JS + a vendored Sveltia bundle. No `npm install`,
no bundler. You just need a static file server and a browser.

---

## 1. Local development & testing

### 1.1 Serve the files

A service worker + manifest only work over `http(s)`, not `file://`. Serve the repo root:

```sh
# Python (already on most machines)
python -m http.server 8123 --bind 127.0.0.1
# open http://127.0.0.1:8123/
```

```powershell
# PowerShell equivalent
python -m http.server 8123 --bind 127.0.0.1
# then browse http://127.0.0.1:8123/
```

> Note the base path. Locally the site is at `/` so everything works. On GitHub **project**
> pages it's `/<repo>/`, which is why all paths in this repo are **relative** — see
> `skill/references/pwa-shell.md`. Test the relative-path behaviour by serving from a
> subfolder if you want to mimic Pages exactly:
> `mkdir -p _site/focknote && cp -r <files> _site/focknote && (cd _site && python -m http.server 8123)`
> then open `http://127.0.0.1:8123/focknote/`.

### 1.2 Smoke test (no browser)

Confirm every shell path is reachable and the admin loads the same-origin bundle:

```sh
for p in / /manifest.json /sw.js /admin/ /admin/sveltia-cms.js \
         /assets/icons/icon-512.png /knowledge/note/welcome.md; do
  printf '%s  %s\n' "$(curl -s -o /dev/null -w '%{http_code}' http://127.0.0.1:8123$p)" "$p"
done
curl -s http://127.0.0.1:8123/admin/ | grep -o 'src="[^"]*"'   # expect ./sveltia-cms.js
```

Validate the config files after editing them:

```sh
python -c "import json; json.load(open('manifest.json')); print('manifest OK')"
python -c "import yaml; yaml.safe_load(open('admin/config.yml')); print('config OK')"
```

### 1.3 Editor / token round-trip (real GitHub)

The editor (`/admin/`) talks to the **live GitHub API** — there is no local notes backend.
To exercise editing locally:

1. Point `admin/config.yml` `backend.repo` at a real repo you own (a throwaway is fine).
2. Serve and open `http://127.0.0.1:8123/admin/`.
3. **Sign in with Token**, paste a short-lived PAT (see `skill/references/token-scopes.md`).
4. Create/edit/delete a note → confirm a commit lands in that repo's `knowledge/note/`.
5. Check accents + emoji survive a reload (`caffè 🔥`).

> Sveltia allows `http://localhost` / `127.0.0.1` as an auth origin, so token login works
> locally. If a browser blocks the popup, allow popups for the localhost origin.

### 1.4 PWA / installability checks

In Chrome/Edge DevTools:

- **Application → Manifest** → no errors, shows "Installable", icons render.
- **Application → Service Workers** → `sw.js` is *activated*, scope is the site root.
- **Lighthouse → PWA** audit → passes (manifest, SW with fetch handler, 512 icon).
- **Offline test:** load once, tick *Offline* in DevTools → Network, reload → the editor
  shell still loads (data sync resumes when back online).

### 1.5 After changing the Sveltia bundle

Re-vendor + bust the cache (see `VENDOR.md`):

```sh
curl -s https://registry.npmjs.org/@sveltia/cms/latest | grep -o '"version":"[^"]*"'
curl -sL https://unpkg.com/@sveltia/cms@<VERSION>/dist/sveltia-cms.js -o admin/sveltia-cms.js
# then bump the version in VENDOR.md AND the CACHE constant in sw.js
```

The `CACHE` bump matters: without it the old bundle stays cached for returning visitors.

### 1.6 Regenerate icons

Icons are generated with Pillow (`pip install pillow`). The generator lives in the build
history; to redraw, edit `assets/icons/favicon.svg` for the design and re-run a Pillow
script that rasterizes the same fork glyph to `icon-512.png` and `icon-192-maskable.png`
(maskable uses ~0.78 scale for the safe zone). Keep both PNGs RGBA.

---

## 2. Running the skill in chat (Tier B)

Use this when you're in **claude.ai chat** (GitHub connector is read-only — Claude can't
create repos or write files, so it guides you through clicks).

### Setup

The skill lives in `skill/SKILL.md` with refs under `skill/references/`. In a chat that has
this repo connected (or paste `SKILL.md`), ask:

> "Set up my FockNote notebook."

### What happens

Claude walks you through, emitting links + exact text to paste. **Default = public mode**
(one repo, one click). Roughly:

1. **Use this template** → create your `focknote` repo (Public).
2. Edit `admin/config.yml` → set `repo:` to `<you>/focknote`. Commit.
3. **Settings → Pages** → Deploy from branch `main`, `/ (root)`.
4. Open `https://<you>.github.io/focknote/`, install, **Sign in with Token**, edit.

For **private mode** Claude adds a step: create a second **private** repo
(`focknote-notes`) and point `repo:` at it. See §B of `skill/SKILL.md`.

### Testing Tier B

- Run it against a throwaway GitHub account.
- Walk each emitted step; confirm the hand-off URL goes live and token login round-trips.
- Confirm Claude *reads* an existing FockNote repo to help draft/organize notes.

---

## 3. Running the skill in Claude Code (Tier A)

Use this in **Claude Code** (local or web/cloud) where Claude has `gh`/git — it does
**everything** and hands back a live URL. **Default = private mode** (two repos).

### Prerequisites

- **Local:** `gh auth login` (a token with repo-create + Pages rights).
- **Cloud (Claude Code on the web):** set `GITHUB_TOKEN` env var + Full network access, then
  `apt-get install -y gh && echo "$GITHUB_TOKEN" | gh auth login --with-token`. Full steps:
  `skill/references/running-in-cloud.md`.

### Run

> "Use the FockNote skill to deploy my notebook in private mode."

Claude executes §A of `skill/SKILL.md`:

1. `gh repo create <you>/focknote --template OWNER/focknote --public --clone`
2. (private) `gh repo create <you>/focknote-notes --private` + seed `main`
3. Write `admin/config.yml` `backend.repo` → the notes repo
4. Commit/push; enable Pages via `gh api .../pages`
5. Run the **verification checklist** (§C of `SKILL.md`)
6. Print the hand-off (URL + install + token steps)

### Testing Tier A

Dry-run the mechanics without touching production:

```sh
gh auth status                         # confirm write access
gh repo create demo-focknote --template <you>/focknote --public --clone   # use your own template fork
# inspect the clone, run the §1.2 smoke test against the cloned files
gh api repos/<you>/demo-focknote/pages 2>/dev/null || echo "pages not enabled yet"
```

Then run the skill's verification checklist by hand:

- Pages URL → 200, FockNote landing.
- `…/manifest.json` valid, 192 + 512 icons.
- `/admin/` loads same-origin `./sveltia-cms.js`.
- `config.yml` `backend.repo` = the intended (private) repo.
- SW scope = `/<repo>/`.
- **Private mode:** `…github.io/<repo>/knowledge/note/welcome.md` → **404** (notes aren't on
  Pages).
- Token login → create note → commit in the notes repo → survives reload; accents/emoji OK.

Delete the demo repos when done: `gh repo delete <you>/demo-focknote --yes`.

---

## Quick reference

| Task | Command |
|------|---------|
| Serve locally | `python -m http.server 8123 --bind 127.0.0.1` |
| Validate manifest | `python -c "import json;json.load(open('manifest.json'))"` |
| Validate config | `python -c "import yaml;yaml.safe_load(open('admin/config.yml'))"` |
| Latest Sveltia | `curl -s https://registry.npmjs.org/@sveltia/cms/latest \| grep -o '"version":"[^"]*"'` |
| Re-vendor bundle | `curl -sL https://unpkg.com/@sveltia/cms@<V>/dist/sveltia-cms.js -o admin/sveltia-cms.js` |
| Skill (chat) | "Set up my FockNote notebook" → Tier B wizard |
| Skill (Claude Code) | "Use the FockNote skill, private mode" → Tier A auto |
