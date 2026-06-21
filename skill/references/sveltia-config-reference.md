# Sveltia config reference (FockNote)

Full upstream docs: https://sveltiacms.app/en/docs/ — Sveltia is Decap-config-compatible.

## The full notebook `config.yml`

`knowledge/` follows the [Open Knowledge Format](https://github.com/focknote/focknote/blob/main/knowledge/reference/MEMORY.md)
(OKF): one folder per `type`, every file carries a `type:` field. The template
ships five collections, one per type — `note` shown here, the rest
(`project`/`reference`/`decision`/`log`) repeat the same shape against their own
folder + default:

```yaml
backend:
  name: github
  repo: OWNER/NOTES_REPO   # owner/repo that HOLDS THE NOTES
  branch: main
  # no base_url -> built-in "Sign in with Token" (PAT) login

media_folder: content/media
public_folder: /content/media

collections:
  - name: note
    label: Notes
    label_singular: Note
    folder: knowledge/note
    create: true
    delete: true
    format: frontmatter
    extension: md
    slug: "{{year}}-{{month}}-{{day}}-{{slug}}"
    sortable_fields: [date, title]
    summary: "{{title}}"
    fields:
      - { name: type,  label: Type,  widget: hidden, default: note }
      - { name: title, label: Title, widget: string }
      - { name: date,  label: Date,  widget: datetime, default: "" }
      - { name: tags,  label: Tags,  widget: list, required: false }
      - { name: body,  label: Body,  widget: markdown }
  # project / reference / decision / log: same fields, folder: knowledge/<type>,
  # type default: <type>
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
| `folder` | `knowledge/<type>` | Folder collection: one Markdown file per note, one folder per OKF type. |
| `type` | hidden, defaulted per collection | The only OKF-mandatory field — picks the folder, makes every file self-describing. |
| `format: frontmatter` | — | YAML frontmatter + Markdown body. |
| `slug` | date-prefixed (`note`) or plain (others) | Stable, sortable filenames. |
| `fields` | type/title/date/tags/body | The note shape. `body` is the Markdown widget (the editor). |

## Mode → `repo` value

- **Private mode:** `repo: <you>/focknote-notes` (the separate private repo).
- **Public mode:** `repo: <you>/focknote` (this same public repo).

## NOT enabled (on purpose)

- `publish_mode: editorial_workflow` — **unimplemented in Sveltia** as of 2026 (planned
  before its 1.0). Adding it does nothing useful yet. Team review today = fork → PR via the
  GitHub UI. Revisit when Sveltia ships it.

## Extending later (fast-follows)

- Add another `knowledge/<type>/` collection for a new OKF type.
- Add fields (e.g. `draft: boolean`, `cover: image`).
- i18n: Sveltia has first-class multi-locale support if you want bilingual notes.
