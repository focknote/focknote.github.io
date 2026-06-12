# Optional: one-click "Login with GitHub" (Cloudflare Worker + OAuth app)

**Not required.** MVP uses Sveltia's built-in token paste. This replaces that with a
"Login with GitHub" button — nicer for non-technical users. It adds infrastructure (a free
Cloudflare Worker + a GitHub OAuth app). The long-term endgame is **client-side PKCE**,
which removes the Worker entirely once GitHub ships it for browser apps.

## What you're building

GitHub's OAuth web flow needs a server-side step to exchange the code for a token (the
client secret can't live in the browser). Sveltia provides a ready-made Worker for exactly
this: **`sveltia/sveltia-cms-auth`**.

## Steps

1. **GitHub OAuth app** — https://github.com/settings/developers → *New OAuth App*:
   - Homepage URL: your Pages URL (`https://<you>.github.io/focknote/`).
   - Authorization callback URL: your Worker URL + `/callback` (set after step 2).
   - Note the **Client ID** and generate a **Client Secret**.
2. **Deploy the Worker** — fork/deploy `sveltia/sveltia-cms-auth` to Cloudflare Workers
   (free tier). Set env vars `GITHUB_CLIENT_ID`, `GITHUB_CLIENT_SECRET`, and
   `ALLOWED_DOMAINS` (your Pages host). Copy the deployed Worker URL back into the OAuth
   app's callback (`https://<worker>/callback`).
3. **Point Sveltia at the Worker** — in `admin/config.yml`:
   ```yaml
   backend:
     name: github
     repo: OWNER/NOTES_REPO
     branch: main
     base_url: https://<your-worker-subdomain>.workers.dev
   ```
4. Reload `/admin/` → the login screen now shows **Login with GitHub**. Token paste still
   works as a fallback.

Reference: https://github.com/sveltia/sveltia-cms-auth

## Migration note (PKCE)

When GitHub ships PKCE for the browser OAuth flow, the Worker becomes unnecessary: the app
does the code exchange client-side and you can delete the Worker + secret. Watch the
Sveltia roadmap.
