# jj-jujutsu Agent Skill

`jj-jujutsu` teaches agents how to use Jujutsu (`jj`) for version-control
workflows instead of raw Git commands. It covers status, diffs, committing,
file-specific commits, moving or removing files from commits, history rewriting,
bookmarks, Git remote interop, workspaces, conflicts, and recovery.

## Repository Layout

```text
.
├── README.md
└── skills/
    └── jj-jujutsu/
        ├── SKILL.md
        ├── config.md
        ├── git-interop.md
        ├── history.md
        ├── pager-commands.md
        ├── revsets.md
        └── sharing.md
```

The installable skill lives at `skills/jj-jujutsu/`. The `SKILL.md`
frontmatter uses `name: jj-jujutsu`, matching the skill directory name.

## Install With GitHub CLI

Replace `OWNER/REPO` with the GitHub repository that hosts this skill.

```bash
gh skill install OWNER/REPO jj-jujutsu --agent codex --scope user
```

For faster installation from a larger repository, install by path:

```bash
gh skill install OWNER/REPO skills/jj-jujutsu --agent codex --scope user
```

To install into a project instead of user scope:

```bash
gh skill install OWNER/REPO jj-jujutsu --agent codex --scope project
```

## Install With npx skills

Install globally for Codex:

```bash
npx skills add OWNER/REPO --skill jj-jujutsu -a codex -g -y
```

List skills discoverable in the repository:

```bash
npx skills add OWNER/REPO --list
```

Install from the direct GitHub skill path:

```bash
npx skills add https://github.com/OWNER/REPO/tree/main/skills/jj-jujutsu -a codex -g -y
```

## Local Development Install

From the repository root:

```bash
gh skill install . jj-jujutsu --from-local --agent codex --scope user
```

Or with `npx skills`:

```bash
npx skills add . --skill jj-jujutsu -a codex -g
```

If the skill does not appear immediately in Codex, restart or reload Codex so it
rescans installed skills.

## Validation

Before publishing, verify the repository exposes one installable skill:

```bash
find skills -name SKILL.md
```

Optional checks when the CLIs are available:

```bash
gh skill publish --dry-run
npx skills add . --list
```
