# What is FockNote

**FockNote** is a free, installable, git-backed personal notebook — "fork your own Notion."
Every note is a Markdown file in **your** GitHub repo, which makes it both your notebook
**and** a shared workspace + memory that you and **Claude** read and write. The main surface
is **`/read/`** — a clean, installable read-and-edit-in-place app: wiki-links + backlinks,
search, callouts, a slash menu, daily notes, all editing in place with no separate form.
[Sveltia CMS](https://sveltiacms.app/) (`/admin/`) stays available as a fallback for bulk
edits. Capture from any of these, or from Claude Code / Claude chat (GitHub connector).
GitHub Pages hosts the app shell; git is the whole backend. No server, no subscription, no
lock-in.

> 🧠 **Notes as a shared brain.** Drop in the memory bridge (`knowledge/reference/` +
> `CLAUDE.md`) and Claude treats your notebook as project memory — curating the best of your
> sessions back as commits you can see, edit, and share. See
> [INTERFACE.md](https://github.com/focknote/focknote/blob/main/INTERFACE.md).
>
> The notebook's `knowledge/` tree follows Google's [Open Knowledge
> Format](https://github.com/focknote/focknote/blob/main/knowledge/reference/MEMORY.md)
> (OKF) — Markdown + YAML frontmatter, one folder per `type`.

> 🍴 The name is "ForkNote" in a thick accent. The fork is the git kind. The cheek is the point.

This site is the **showcase + docs + the setup skill**. The notebook itself lives in the
template repo: **[focknote/focknote](https://github.com/focknote/focknote)**.

## Quick setup

The fastest path is the **FockNote skill** for Claude:

1. **[Install the skill](skill/references/installing-the-skill.md)** — in Claude Code:
   `/plugin marketplace add focknote/focknote.github.io` then
   `/plugin install focknote@focknote` (or point chat at it).
2. Ask Claude *"stand up my FockNote notebook"* — the [skill](skill/SKILL.md) creates the
   repo(s), wires the config, enables Pages, and hands you a live URL.

Prefer to do it by hand? Manual path:

1. **[Use this template](https://github.com/focknote/focknote/generate)** → create your
   notebook repo (name it anything).
2. Set `backend.repo` in `admin/config.yml` to the repo that holds your notes.
3. **Settings → Pages →** deploy from branch `main`, `/ (root)`.
4. Open `https://<you>.github.io/<repo>/`, **Install app**, **Sign in with Token**, and write
   in place. The `/admin/` Sveltia form stays available for bulk edits, but the main path is
   the `/read/` read-and-edit app — or let Claude read and write the repo directly.

Saving a note = a git commit. That's it.

## Add it to a repo you already have

Don't need a whole notebook — just want Claude to keep curated, shareable memory in a repo
you already work in (a coding project, a docs repo)? Drop in the **memory bridge**
(`knowledge/reference/` + a thin `CLAUDE.md` import block) and skip the notebook setup
entirely. No Pages, no second repo, no Sveltia. Ask Claude *"add FockNote memory to this
repo"* — see [Add the memory bridge to an existing repo](skill/SKILL.md#add-the-memory-bridge-to-an-existing-repo)
in the skill.

## Two privacy modes

| Mode | Repos | Notes are | Cost |
|------|-------|-----------|------|
| **Private** (default) | a **public** app-shell repo + a separate **private** notes repo | private | $0 |
| **Public** (knowledge garden) | one **public** repo holds shell + notes | public | $0 |

**Why two repos for private mode?** GitHub Pages on the Free plan needs a *public* repo, but
Pages only ever serves the **app shell** (code — no notes). Your notes are read/written by
Sveltia via the **GitHub API with your token**, never served by Pages. So the shell can be
public while the notes repo stays private. Fetch a note's raw path on the Pages domain → 404.
It isn't there.

## Naming your repos

Names are **free** — no convention. The docs use `focknote` / `focknote-notes` as defaults.

- The shell repo name becomes your URL path: `https://<you>.github.io/<name>/` (paths are
  relative, so any name works).
- Name the shell repo **`<you>.github.io`** → it serves at the bare root (no path). One per
  account.
- The notes repo name is arbitrary; just keep `backend.repo` in the config matching it.

## Tokens & privacy

Use a **fine-grained** GitHub token scoped to just your notes repo (*Contents: read/write*,
*Metadata: read*), with a **short expiry**. Full details: [Token scopes](skill/references/token-scopes.md).

## Honest limitations

- **No PR-per-save yet** — Sveltia hasn't implemented `editorial_workflow` (planned before
  its 1.0). MVP commits to `main`; team review = fork → PR via the GitHub UI.
- **One-click "Login with GitHub"** (Cloudflare Worker + OAuth) is an optional upgrade, not
  required — see [the OAuth guide](skill/references/oauth-worker-setup.md).
- **Offline is partial.** The app shell + editor load with no network, but you can't
  *edit or browse notes offline*: GitHub is the backend, and Sveltia re-checks auth against
  GitHub on startup, so offline it shows a login screen. This is a Sveltia design boundary —
  a local-only backend was [declined upstream](https://github.com/sveltia/sveltia-cms/issues/630)
  ("local data is stored in IndexedDB, partitioned by repository"). True offline *reading*
  would come via the static reading-view fast-follow below.
- A clean **reading view** (`/read/`, rendered Markdown) ships today; `[[backlinks]]` +
  search are the next fast-follow. (Reading still needs a token round-trip in private mode,
  so it isn't fully offline yet.)

## Contributing

See [Developing & testing](DEVELOPING.md). The template lives in
[focknote/focknote](https://github.com/focknote/focknote); this repo is the site + skill.
