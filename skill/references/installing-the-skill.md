# Installing the skill

The FockNote skill is just `SKILL.md` + the `references/` folder. "Installing" it means
putting those files where a Claude can discover them, so you can say *"set up my FockNote
notebook"* and it runs. Pick the path that matches where you'll use it.

## Claude Code (local) — recommended

Claude Code discovers skills in a `skills/` folder. Drop FockNote's in:

```sh
git clone https://github.com/focknote/focknote.github.io fn
mkdir -p ~/.claude/skills/focknote
cp -r fn/skill/SKILL.md fn/skill/references ~/.claude/skills/focknote/
```

```powershell
git clone https://github.com/focknote/focknote.github.io fn
New-Item -ItemType Directory -Force "$HOME\.claude\skills\focknote" | Out-Null
Copy-Item fn\skill\SKILL.md, fn\skill\references "$HOME\.claude\skills\focknote\" -Recurse
```

Then in Claude Code, invoke **`/focknote`** or just ask *"stand up my FockNote notebook"*.
The `name:` / `description:` in `SKILL.md`'s frontmatter is what makes it discoverable.

> Project-scoped instead of personal? Put it at `<your-repo>/.claude/skills/focknote/` and
> commit it — anyone working in that repo gets the skill.

## Claude Code on the web (cloud)

A cloud session reads skills committed to the repo it's working in. In that repo add:

```
.claude/skills/focknote/SKILL.md
.claude/skills/focknote/references/…
```

(copied from this repo's `skill/`). Commit, push, open the repo in Claude Code web, and ask
it to run the skill. Also do the token/network setup in
[Running in the cloud](running-in-cloud.md) (`GITHUB_TOKEN` env + Full network +
`apt-get install -y gh`).

## Claude.ai chat (Tier B, guided)

You can't install skills in chat. Two ways to still use it:

- **Connect this repo** with the GitHub connector so Claude can read
  `skill/SKILL.md` directly, then ask it to follow the skill, **or**
- **Paste** the contents of
  [`skill/SKILL.md`](https://focknote.github.io/skill/SKILL.md) into the chat and ask Claude
  to follow it.

Either way Claude runs **Tier B** (it emits links + text for you to click/paste).

## No install (one-off)

Point any Claude at the raw skill and tell it to follow:

- Rendered: https://focknote.github.io/skill/SKILL.md
- Raw: https://raw.githubusercontent.com/focknote/focknote.github.io/main/skill/SKILL.md

## Updating the skill

Re-copy `SKILL.md` + `references/` from this repo over your installed copy. The skill is
plain Markdown — no build step.
