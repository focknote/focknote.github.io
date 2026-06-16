---
name: focknote
description: >-
  Stand up a FockNote — a free, installable, git-backed Markdown notebook on GitHub Pages
  that doubles as shared workspace + memory for you and Claude (PWA shell + in-browser
  Sveltia editor). Use when the user wants to create/deploy/scaffold their own notebook,
  "fork your own Notion", a private notes app on GitHub, a Claude-readable notes/memory repo,
  or to update/re-vendor an existing FockNote. Goes from nothing to a deployed notebook +
  hand-off URL.
---

# FockNote setup skill

Goal: **one request → a deployed, installable, editable notebook**, then a hand-off the
user can act on. Notes are Markdown committed to the user's git repo — read/written by
Claude (Code or chat), the in-app reading view, or the in-browser Sveltia form. GitHub
Pages hosts the app shell. Optionally wire the **memory bridge** so the notebook doubles
as Claude's project memory (see "Add the memory bridge to an existing repo").

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
# seed the PRIVATE notes repo with the content folder + the memory bridge...
git clone https://github.com/<you>/focknote-notes ../focknote-notes
cp -r content CLAUDE.md ../focknote-notes/ && (cd ../focknote-notes && git add -A \
  && git commit -m "Seed notes + memory bridge" && git push -u origin main)
# ...then DELETE notes + bridge from the PUBLIC shell so they're never served by Pages
git rm -r content CLAUDE.md && git commit -m "Private mode: notes live in the private repo"
```
> **Why the delete matters:** Sveltia reads notes from `backend.repo` via the GitHub API,
> not from the shell's `content/` folder. If `content/` stays in the public shell, the
> welcome note is reachable at `…github.io/focknote/content/notes/welcome.md` (200) — a
> privacy leak. In private mode the public shell must contain **no notes**. (Public mode
> keeps `content/` — there the shell *is* the notes repo.)

**Memory bridge.** The template ships three files that make the notebook double as
Claude's project memory: root `CLAUDE.md` (a thin marker-delimited block that
**imports** `@content/agent/MEMORY.md`), `content/agent/MEMORY.md` (the read/write
conventions — single source of truth), and `content/agent/INDEX.md` (the index
note). When Claude Code opens the notes repo, the harness loads `CLAUDE.md` → the
import pulls in the conventions → Claude treats `content/agent/` (`INDEX.md` first)
as the authoritative store. The seed copies all three into the **notes** repo (not
the public shell). The root file is a thin *import* on purpose: it makes adding the
bridge to an **existing** repo a non-destructive append — see "Add the memory bridge
to an existing repo" below. (Concept: `INTERFACE.md`.)

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

**B1c. (Memory bridge — optional)** "To let Claude use this notebook as its memory,
add three files to the **notes** repo (copy each from
**https://github.com/OWNER/focknote**): `CLAUDE.md` (root), `content/agent/MEMORY.md`,
and `content/agent/INDEX.md`. Claude Code then treats `content/agent/` as its store.
Adding to a repo that *already* has a `CLAUDE.md`? Don't replace it — paste only the
`<!-- focknote:memory:start … end -->` block to the **end** of your existing file."

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
- [ ] **Memory bridge:** the **notes** repo root has `CLAUDE.md` and
      `content/agent/INDEX.md` (private mode: these are NOT in the public shell).

If a step fails → [`references/troubleshooting.md`](references/troubleshooting.md).

## §A7. Hand-off text (print this to the user)

> ✅ **Your FockNote is live:** `https://<you>.github.io/focknote/`
>
> 1. Open that link on your phone. Tap **Install app** (or browser menu → *Add to Home
>    screen*).
> 2. Open the app → it lands on your notes (the `/read/` reading view). Tap a note to
>    read; ✎ to edit in place; or **Edit in CMS** for the full Sveltia form.
> 3. Click **Sign in with Token**, follow the link, generate a GitHub token, paste it.
> 4. Edit the welcome note (or delete it and write your own). **Saving = a git commit.**
>
> 🔐 **Token note:** the pre-filled link uses a broad classic `repo` token (all repos,
> read/write). Tighter option: a **fine-grained** token scoped to just your notes repo
> (*Contents: read/write*, *Metadata: read*). Use a **short expiry**. Details:
> `skill/references/token-scopes.md`.

## Add the memory bridge to an existing repo

Use when the user wants Claude-managed memory **in a repo they already have** (a
coding project, a docs repo) without standing up a whole notebook — e.g. "store the
best of my Claude sessions here to share with friends." This drops in a
`content/agent/` folder + wires `CLAUDE.md` **non-destructively**.

> **Conflict warning — never overwrite `CLAUDE.md`.** An existing repo usually
> already has a `CLAUDE.md` of coding instructions. Do **not** clobber it. The
> bridge is a marker-delimited block (`<!-- focknote:memory:start … end -->`) that
> only **imports** `@content/agent/MEMORY.md`, so it appends cleanly and is
> idempotent. Coding instructions stay intact; memory layers on top.

```sh
SRC=path/to/focknote      # a clone of the focknote template (has content/agent/ + CLAUDE.md)
DEST=path/to/your/repo    # the existing repo to add memory to

mkdir -p "$DEST/content/agent"
cp "$SRC/content/agent/MEMORY.md" "$SRC/content/agent/INDEX.md" "$DEST/content/agent/"

# Wire CLAUDE.md: append the marker block, never overwrite. Idempotent.
if [ -f "$DEST/CLAUDE.md" ] && grep -q "focknote:memory:start" "$DEST/CLAUDE.md"; then
  echo "memory bridge already present — nothing to do"
else
  [ -s "$DEST/CLAUDE.md" ] && printf '\n' >> "$DEST/CLAUDE.md"  # separate from existing content
  cat "$SRC/CLAUDE.md" >> "$DEST/CLAUDE.md"                     # the focknote:memory block (creates file if absent)
fi
# then: cd "$DEST" && git add content/agent CLAUDE.md && git commit && git push
```

To **remove** later: delete the lines between the two `focknote:memory` markers and
the `content/agent/` folder. (Tier B: have the user paste the marker block from
`OWNER/focknote/blob/main/CLAUDE.md` to the end of their existing `CLAUDE.md`, and add
the two `content/agent/` files by hand.)

## Updating an existing FockNote

Re-vendor Sveltia (newest stable) + bump the SW cache: follow [VENDOR.md](https://github.com/focknote/focknote/blob/main/VENDOR.md),
commit, push. Pages redeploys automatically.

## Optional: one-click "Login with GitHub"

Not needed for MVP. To replace token-paste with an OAuth button, set up the Cloudflare
Worker + GitHub OAuth app per
[`references/oauth-worker-setup.md`](references/oauth-worker-setup.md).
