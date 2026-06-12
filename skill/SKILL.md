---
name: focknote
description: >-
  Stand up a FockNote — a free, installable, git-backed personal notebook (Sveltia CMS
  + PWA on GitHub Pages). Use when the user wants to create/deploy/scaffold their own
  notebook, "fork your own Notion", a private notes app on GitHub, or to update/re-vendor
  an existing FockNote. Goes from nothing to a deployed, editable notebook + hand-off URL.
---

# FockNote setup skill

Goal: **one request → a deployed, installable, editable notebook**, then a hand-off the
user can act on. Notes are Markdown committed to the user's git repo; Sveltia CMS is the
editor; GitHub Pages hosts the app shell.

## 0. Pick the execution tier (decide first)

You can only fully automate this where you have **GitHub write access**:

- **Tier A — Claude Code (local or web/cloud): automate everything.** You have `gh`/git.
  Default to **private mode**. Proceed through §A.
- **Tier B — claude.ai chat (read-only connector): you can't write.** Become a guided
  wizard: emit links + exact text the user clicks/pastes. Default to **public mode**
  (one-click). Proceed through §B.

If unsure which you are: try `gh auth status` (or `git`/Bash). If it works → Tier A. If you
have no shell/write tools → Tier B.

Confirm **mode** with the user if they haven't said:
- **Private** (default): public app-shell repo + separate **private** notes repo. Personal.
- **Public**: one public repo, notes shared on purpose. Knowledge garden.

Also **ask the user what to name the repo(s)** — names are free, not a convention. Defaults
below use `focknote` / `focknote-notes`; substitute whatever they choose. Two facts to tell
them:
- The shell repo name becomes the URL path: `https://<you>.github.io/<name>/` (relative
  paths mean any name works).
- Naming the shell repo **`<you>.github.io`** serves it at the root `https://<you>.github.io/`
  (no path) — one per account.
Whatever they pick, `backend.repo` in the config must match the **notes** repo's real name
(the skill writes that line, so keep them in sync).

---

## §A. Tier A — automated (Claude Code)

Prereq: a GitHub token with repo-create + Pages perms. Locally that's `gh auth login`.
In Claude Code **web/cloud** you must set it yourself — see
[`references/running-in-cloud.md`](references/running-in-cloud.md) (custom env
`GITHUB_TOKEN` + Full network + `apt-get install -y gh`). Get the username:
`gh api user -q .login`.

### A1. Create the repo(s)

Template repo = `OWNER/focknote` (this template). Replace `<you>` with the user's login.

**Public mode** (one repo):
```sh
gh repo create <you>/focknote --template OWNER/focknote --public --clone
```

**Private mode** (default — two repos):
```sh
# 1) public app-shell repo from the template
gh repo create <you>/focknote --template OWNER/focknote --public --clone
# 2) separate PRIVATE notes repo
gh repo create <you>/focknote-notes --private
```
Then **move the notes out of the public shell into the private repo** — this is what makes
private mode actually private:
```sh
cd focknote
# seed the PRIVATE notes repo with the content folder...
git clone https://github.com/<you>/focknote-notes ../focknote-notes
cp -r content ../focknote-notes/ && (cd ../focknote-notes && git add -A \
  && git commit -m "Seed notes" && git push -u origin main)
# ...then DELETE content/ from the PUBLIC shell so notes are never served by Pages
git rm -r content && git commit -m "Private mode: notes live in the private repo"
```
> **Why the delete matters:** Sveltia reads notes from `backend.repo` via the GitHub API,
> not from the shell's `content/` folder. If `content/` stays in the public shell, the
> welcome note is reachable at `…github.io/focknote/content/notes/welcome.md` (200) — a
> privacy leak. In private mode the public shell must contain **no notes**. (Public mode
> keeps `content/` — there the shell *is* the notes repo.)

### A2. Write the config

Edit `admin/config.yml` in the **app-shell** clone → set `backend.repo`:
- private mode → `<you>/focknote-notes`
- public mode → `<you>/focknote`

Leave `branch: main`, no `base_url`. (Field reference:
[`references/sveltia-config-reference.md`](references/sveltia-config-reference.md).)

### A3. Confirm the vendored bundle

`admin/sveltia-cms.js` ships pinned in the template. To ship the newest stable instead,
re-vendor per [VENDOR.md](https://github.com/focknote/focknote/blob/main/VENDOR.md) and bump `CACHE` in `sw.js`.

### A4. Brand + seed

The template is already branded **FockNote** (`manifest.json` name/short_name/description
= "Own your fockin' notes.", icons, welcome note). Personalize title/seed note if the user
named their notebook. Commit + push the app-shell repo.

### A5. Enable GitHub Pages (app-shell repo)

```sh
gh api -X POST repos/<you>/focknote/pages -f "source[branch]=main" -f "source[path]=/" \
  || gh api -X PUT repos/<you>/focknote/pages -f "source[branch]=main" -f "source[path]=/"
```
Pages URL: `https://<you>.github.io/focknote/`. First build takes ~1 min.

### A6. Verify (run the checklist in §C), then A7 hand-off.

---

## §B. Tier B — guided wizard (claude.ai chat)

You can't create repos or write files. **Emit** these for the user, in order. Default to
**public mode**; offer private mode (it costs the extra repo in step B1b).

**B1a. Create the repo.** "Open **https://github.com/OWNER/focknote** → click **Use this
template → Create a new repository**. Name it `focknote`. Pick **Public**. Create it."

**B1b. (Private mode only)** "Also create an empty **Private** repo at
**https://github.com/new** named `focknote-notes`, **Add a README** (so it has a `main`
branch)."

**B2. Set the config.** "In your new `focknote` repo, open `admin/config.yml`, click edit,
and set the `repo:` line to **`<you>/focknote-notes`** (private mode) or **`<you>/focknote`**
(public mode). Commit." Paste them the exact full `config.yml` if helpful (from
[`references/sveltia-config-reference.md`](references/sveltia-config-reference.md)).

**B3. Enable Pages.** "Open **`https://github.com/<you>/focknote/settings/pages`** → Source
= **Deploy from a branch** → Branch **`main`**, folder **`/ (root)`** → Save."

**B4. Hand off** (§A7 text). You can also *draft note content* for them to paste into the
editor, and *read* an existing FockNote repo to help organize it.

---

## §C. Verification checklist (run before hand-off)

- [ ] Pages URL returns **200** and shows the FockNote landing page.
- [ ] `https://<you>.github.io/focknote/manifest.json` is valid JSON with name,
      short_name, start_url, display=standalone, theme/background color, and 192+512 icons.
- [ ] `/admin/` loads; its `<script src>` is the **same-origin** `./sveltia-cms.js`
      (not a CDN URL).
- [ ] `admin/config.yml` `backend.repo` = the intended repo (private notes repo in
      private mode).
- [ ] Service worker registers; its scope is `/focknote/`.
- [ ] **Private mode privacy:** `https://<you>.github.io/focknote/content/notes/welcome.md`
      returns **404** (notes aren't on Pages — they're in the private repo).
- [ ] Token login round-trips: create a note → it appears as a commit in the notes repo's
      `content/notes/`, survives reload; accents + emoji intact.

If a step fails → [`references/troubleshooting.md`](references/troubleshooting.md).

## §A7. Hand-off text (print this to the user)

> ✅ **Your FockNote is live:** `https://<you>.github.io/focknote/`
>
> 1. Open that link on your phone. Tap **Install app** (or browser menu → *Add to Home
>    screen*).
> 2. Open the app → it lands on the editor (`/admin/`).
> 3. Click **Sign in with Token**, follow the link, generate a GitHub token, paste it.
> 4. Edit the welcome note (or delete it and write your own). **Saving = a git commit.**
>
> 🔐 **Token note:** the pre-filled link uses a broad classic `repo` token (all repos,
> read/write). Tighter option: a **fine-grained** token scoped to just your notes repo
> (*Contents: read/write*, *Metadata: read*). Use a **short expiry**. Details:
> `skill/references/token-scopes.md`.

## Updating an existing FockNote

Re-vendor Sveltia (newest stable) + bump the SW cache: follow [VENDOR.md](https://github.com/focknote/focknote/blob/main/VENDOR.md),
commit, push. Pages redeploys automatically.

## Optional: one-click "Login with GitHub"

Not needed for MVP. To replace token-paste with an OAuth button, set up the Cloudflare
Worker + GitHub OAuth app per
[`references/oauth-worker-setup.md`](references/oauth-worker-setup.md).
