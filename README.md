# What is FockNote

**FockNote** is a free, installable, git-backed personal notebook — "fork your own Notion."
Every note is a Markdown file committed to **your** GitHub repo. The editor
([Sveltia CMS](https://sveltiacms.app/)) runs entirely in your browser; GitHub Pages hosts
the app shell; git is the whole backend. No server, no subscription, no lock-in.

> 🍴 The name is "ForkNote" in a thick accent. The fork is the git kind. The cheek is the point.

This site is the **showcase + docs + the setup skill**. The notebook itself lives in the
template repo: **[focknote/focknote](https://github.com/focknote/focknote)**.

## Quick setup

The fastest path is to ask **Claude** to run the [setup skill](skill/SKILL.md) — it creates
the repo(s), wires the config, enables Pages, and hands you a live URL. Manual path:

1. **[Use this template](https://github.com/focknote/focknote/generate)** → create your
   notebook repo (name it anything).
2. Set `backend.repo` in `admin/config.yml` to the repo that holds your notes.
3. **Settings → Pages →** deploy from branch `main`, `/ (root)`.
4. Open `https://<you>.github.io/<repo>/`, **Install app**, open the editor, **Sign in with
   Token**, and write.

Saving a note = a git commit. That's it.

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
- A polished **reading view** with rendered Markdown + `[[backlinks]]` is a planned
  fast-follow; today the installable app is the editor. It would also enable offline reading.

## Contributing

See [Developing & testing](DEVELOPING.md). The template lives in
[focknote/focknote](https://github.com/focknote/focknote); this repo is the site + skill.
