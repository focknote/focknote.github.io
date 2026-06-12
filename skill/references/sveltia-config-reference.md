# Sveltia config reference (FockNote)

Full upstream docs: https://sveltiacms.app/en/docs/ — Sveltia is Decap-config-compatible.

## The full notebook `config.yml`

```yaml
backend:
  name: github
  repo: OWNER/NOTES_REPO   # owner/repo that HOLDS THE NOTES
  branch: main
  # no base_url -> built-in "Sign in with Token" (PAT) login

media_folder: content/media
public_folder: /content/media

collections:
  - name: notes
    label: Notes
    label_singular: Note
    folder: content/notes
    create: true
    delete: true
    format: frontmatter
    extension: md
    slug: "{{year}}-{{month}}-{{day}}-{{slug}}"
    sortable_fields: [date, title]
    summary: "{{title}}"
    fields:
      - { name: title, label: Title, widget: string }
      - { name: date,  label: Date,  widget: datetime, default: "" }
      - { name: tags,  label: Tags,  widget: list, required: false }
      - { name: body,  label: Body,  widget: markdown }
```

## Field-by-field

| Key | Value | Why |
|-----|-------|-----|
| `backend.name` | `github` | GitHub API backend. |
| `backend.repo` | `owner/repo` | **The notes repo.** Can differ from the repo hosting `/admin/` — it's an explicit API target, not inferred from the host. This is what makes private mode work. |
| `backend.branch` | `main` | Single working branch (Sveltia is single-branch today). |
| *(no `base_url`)* | — | Omitting it = PAT login. Add it only for the optional OAuth Worker. |
| `media_folder` | `content/media` | Where uploads are committed (in the notes repo). |
| `public_folder` | `/content/media` | URL prefix written into note frontmatter for media. |
| `folder` | `content/notes` | Folder collection: one Markdown file per note. |
| `format: frontmatter` | — | YAML frontmatter + Markdown body. |
| `slug` | date-prefixed | Stable, sortable filenames. |
| `fields` | title/date/tags/body | The note shape. `body` is the Markdown widget (the editor). |

## Mode → `repo` value

- **Private mode:** `repo: <you>/focknote-notes` (the separate private repo).
- **Public mode:** `repo: <you>/focknote` (this same public repo).

## NOT enabled (on purpose)

- `publish_mode: editorial_workflow` — **unimplemented in Sveltia** as of 2026 (planned
  before its 1.0). Adding it does nothing useful yet. Team review today = fork → PR via the
  GitHub UI. Revisit when Sveltia ships it.

## Extending later (fast-follows)

- Add a `daily` collection (another folder collection over `content/daily`).
- Add fields (e.g. `draft: boolean`, `cover: image`).
- i18n: Sveltia has first-class multi-locale support if you want bilingual notes.
