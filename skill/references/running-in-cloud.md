# Running the skill in Claude Code on the web (cloud)

Claude Code on the web runs in an Anthropic cloud sandbox that can clone, commit, and push.
By default it can push to repos you've **already connected** — but FockNote's first act is
to **create a brand-new repo**, which isn't connected yet, so the default GitHub App scope
won't cover `gh repo create` or `gh api .../pages`. Fix: give the sandbox its own token.

## One-time setup (custom environment)

1. **Create a GitHub PAT** with repo-create + Pages rights:
   - Classic: scope `repo` (+ it covers Pages). Short expiry.
   - (Fine-grained can't create *new* repos, so for the create step use a classic token,
     or pre-create the repos and use a fine-grained one for the rest.)
2. In Claude Code web → **custom environment**:
   - **Environment variable:** `GITHUB_TOKEN=ghp_xxxx…`
   - **Network access:** **Full** (or Custom allowing `api.github.com`,
     `github.com`, `release-assets.githubusercontent.com`, `unpkg.com`).
3. Make `gh` available + authenticated in the sandbox:
   ```sh
   apt-get install -y gh
   echo "$GITHUB_TOKEN" | gh auth login --with-token
   gh auth status
   ```

Now §A of the skill runs unchanged: `gh repo create --template`, write config, `gh api …
/pages`, push.

## Notes

- Claude Code on the web is a paid-plan research-preview feature (Pro/Max/Team+). The
  *resulting notebook* is free for anyone to use afterward — no Claude needed to edit.
- The same token you put in `GITHUB_TOKEN` can be the one the user later pastes into
  Sveltia, but prefer issuing a separate short-lived token for the scaffolding run.
- Local Claude Code needs none of this — `gh auth login` once and §A works directly.
