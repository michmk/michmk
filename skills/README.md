# Skills

Agent skills for [Claude Code](https://claude.com/claude-code). A skill is a
folder with a `SKILL.md` file. Claude Code reads the file when the task matches
the `description` field in the frontmatter. You can also start a skill by name
with `/<skill-name>`.

## Skills in this folder

| Skill | What it does |
|---|---|
| [`document-changes`](document-changes/SKILL.md) | Documents a set of git changes. It writes and repairs comments, docstrings, and prose docs, and it removes the comment noise that AI-assisted editing leaves behind. The prose follows ASD-STE100 Simplified Technical English. |
| [`code-review`](code-review/SKILL.md) | Reviews a set of git changes as a senior engineer. It summarises the change, gives a short verdict, and lists blockers, warnings, and nits in one table. It looks for silent failures and AI slop. It reports only; it does not edit code. |

## Install

Copy the skill folder to `~/.claude/skills/`. Claude Code finds every skill in
that directory in all your projects.

```bash
cp -R skills/document-changes ~/.claude/skills/
```

To install every skill in this folder:

```bash
cp -R skills/*/ ~/.claude/skills/
```

Restart Claude Code, or start a new session. Run `/help` to confirm that the
skill is in the list.

To install a skill for one project only, copy it to `.claude/skills/` in the
root of that project.

## Use

Claude Code starts a skill on its own when the task matches the description. To
start it yourself, type the folder name with a slash:

```
/document-changes
```

Text after the name goes to the skill as arguments:

```
/document-changes create readme for skills
```
