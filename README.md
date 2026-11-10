# obsidian-project-sync

Keeps the project notes of an [Obsidian](https://obsidian.md) vault in step with the work you do in [Claude Code](https://claude.com/claude-code), without trampling the notes you wrote by hand.

The usual way to have an agent "update the docs" is to let it rewrite the file. In a vault that is curated by hand, that loses prose, footer links and plugin fields a little at a time. This skill works the other way round: Claude drafts, a small script replaces exactly one section, and every write is shown as a diff before it happens.

```
$ python vault.py section --note "Projects/My App/My App.md" --heading "Deploy" --content new.md --dry-run
--- before
+++ after
@@ -14,3 +14,4 @@
 ## 🚀 Deploy

-Deployed by hand from main.
+Deployed by CI on every merge to main, to Cloudflare Pages.
+Previews for every pull request.

[DRY RUN] Nothing was written.
```

## What is in the box

| Path | What it is |
|---|---|
| `obsidian-project-sync/SKILL.md` | The skill: look before writing, read the vault's conventions, replace one section at a time, keep canvases honest |
| `obsidian-project-sync/scripts/vault.py` | Standard-library Python 3.8+: `scan`, `section`, `frontmatter`, `canvas`, `log`, `log-merge` |
| `obsidian-project-sync/references/mechanics.md` | What the script guarantees: config file, heading matching, the protections, the log format |
| `obsidian-project-sync/references/hook-install.md` | Installing the session-logging hook on Windows, macOS and Linux |
| `hook/settings.example.json` | The `settings.json` fragment that wires the hook in |
| `tests/test_vault.py` | The script's tests: `python -m unittest discover -s tests` |

## What it guarantees

- **The vault describes itself.** A `.claude-vault.json` at the vault's root says where projects, finished projects, session logs and the conventions note live. The skill hardcodes no folder names, and says when it is working on fallbacks.
- **The vault's conventions note rules.** The skill carries no copy of your writing rules; it reads the note the config names, and asks before writing when there is none.
- **One section at a time.** Headings are matched without emoji and by partial match. The last section is protected so the footer links below it survive; `append` keeps the footer last and writes inside fenced callouts.
- **Frontmatter: scalars only.** Block lists such as topics are refused; plugin fields are never touched.
- **Canvases that do not lie.** The canvas report flags broken links and cards whose note's `state` no longer matches the group they sit in. New cards land below everything; nothing that exists moves.
- **A session log, not automatic prose.** An optional `SessionEnd` hook records branch, commits and touched files per repository. It never writes in a note: the next time you ask for an update, Claude has the log in front of it.

## Install

Copy the skill folder into your skills directory:

```bash
cp -r obsidian-project-sync ~/.claude/skills/obsidian-project-sync
```

On Windows the folder is `%USERPROFILE%\.claude\skills\obsidian-project-sync`. For claude.ai, zip the `obsidian-project-sync/` folder (the folder itself at the root of the zip) and upload it under **Settings → Capabilities → Skills**; there it writes to the vault through your linked computer when it can, and drafts notes for you to paste when it cannot.

Then describe your vault once, at its root, in `.claude-vault.json` (every key is optional):

```json
{
  "projects": "1 Projects",
  "archive": "4 Archive",
  "logs": "3 Resources/Claude Sync",
  "conventions": "Meta/Conventions.md"
}
```

The hook is optional; `references/hook-install.md` walks through it.

## The local map

The vault's path and anything specific to your own setup live outside the skill, in a short section of your `CLAUDE.md`, so reinstalling or upgrading never loses them:

```markdown
## obsidian-project-sync local map
- Vault: `~/Obsidian/MyVault` (also passed to the hook with `--vault`).
- Every project folder carries a `Decisions.md` note; offer to create it when missing.
- Status callouts: `ad-todo` while open, `ad-done` when finished.
- `~/code/CLAUDE.md` points at project notes: when a note moves, update the pointer.
```

Without a map the skill takes the vault from the `OBSIDIAN_VAULT` environment variable, or asks.

## Using it in claude.ai

claude.ai reaches your vault only while your computer is linked and the vault's folder is connected: with a shell on that computer the skill runs its script there (from a copy already installed on it), with only the file tools it edits the note by hand and checks that each write landed, and without the link it drafts notes for you to paste. It never reads your `CLAUDE.md`, so its local map needs another home. Two ways:

- **In your instructions.** Paste the `## obsidian-project-sync local map` section into a Project's instructions (it applies in that Project only) or into your personal preferences in Settings (it applies in every chat).
- **Inside the package.** Before zipping, add a section named `## Local map for this install` to your copy of `SKILL.md`, just before its first `## ` heading, holding the same lines. The skill reads it as its local map; Project instructions that say otherwise still win. Keep that edited copy outside your clone of this repository and add the section again when you upgrade.

Keep what it needs to write or draft a note: the vault's path on your computer (used only while it is linked), its folder names, its frontmatter schema, the language and voice of your notes, your callout styles. The hook does not run in claude.ai.

## Licence

MIT. See [LICENSE](LICENSE).
