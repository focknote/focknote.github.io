# Token scopes

Sveltia's **Sign in with Token** stores a GitHub personal access token (PAT) in your
browser and uses it to read/write your notes via the GitHub API. Two kinds of PAT work.

## Option 1 — Classic PAT (what the pre-filled link gives you)

The "Sign in with Token" dialog links to GitHub's token page with the **classic `repo`**
scope pre-selected. Quick, but **broad**: `repo` grants read/write to **every** repo your
account can touch. Fine for a throwaway/short-lived token; not ideal long-term.

- Page: https://github.com/settings/tokens/new
- Scope: `repo`
- **Set a short expiry** (e.g. 7–30 days).

## Option 2 — Fine-grained PAT (recommended; required for private notes)

Scope a token to **only your notes repo**. Smallest blast radius, and it's what authorizes
a **private** notes repo.

1. https://github.com/settings/personal-access-tokens/new
2. **Resource owner:** your account.
3. **Repository access:** *Only select repositories* → pick your notes repo
   (e.g. `focknote-notes` in private mode, or `focknote` in public mode).
4. **Permissions → Repository permissions:**
   - **Contents: Read and write** (read/commit/delete notes + media)
   - **Metadata: Read-only** (auto-required)
5. **Expiration:** short.
6. Generate, copy, paste into Sveltia's token prompt.

> If you use editorial review later (fork → PR), the token still only needs access to
> *your own* repo — you never paste a token scoped to someone else's notebook.

## If login fails

- **404 / "not found"** on a private repo → token lacks access to it (wrong repo selected,
  or used a fine-grained token that doesn't include it).
- **403** → missing **Contents: write** (fine-grained) or `repo` scope (classic).
- Token expired → generate a new one; Sveltia will re-prompt.
