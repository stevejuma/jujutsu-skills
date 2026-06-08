---
name: jj-jujutsu
description: Use Jujutsu (`jj`) for version control instead of Git. Use for status, diffs, committing, file-specific commits, moving or removing files from commits, history rewriting, bookmarks, Git remote interop, workspaces, conflicts, and recovery.
license: MIT
compatibility: Requires the `jj` CLI. Optimized for Git-backed or colocated repositories while avoiding direct Git write commands.
metadata:
  version: "0.1.0"
  source: "Merged from the existing jj skill and updated Jujutsu agent guidance."
allowed-tools: Bash(jj:*), Bash(pwd:*), Bash(ls:*), Bash(test:*), Bash(cat:*), Bash(grep:*), Bash(sed:*)
---

# Jujutsu (`jj`) Version Control

Use this skill whenever a repository uses Jujutsu, asks for `jj`, or instructs
agents to avoid Git. Prefer `jj` for all version-control status, diffs, history,
commits, bookmarks, remotes, workspaces, and recovery.

**Tested with jj 0.39.0** - Commands may differ in other versions.

## Reference Map

Load these only when the task needs deeper detail:

| Task | File |
| --- | --- |
| Disable pager or understand which commands page | [pager-commands.md](pager-commands.md) |
| Understand jj/Git interop and Git boundaries | [git-interop.md](git-interop.md) |
| Write revset, fileset, or template expressions | [revsets.md](revsets.md) |
| Push, fetch, manage bookmarks, or work with PRs | [sharing.md](sharing.md) |
| Split, rebase, squash, resolve conflicts, or inspect history | [history.md](history.md) |
| Configure jj, aliases, signing, or diff behavior | [config.md](config.md) |

## Non-Negotiable Agent Rules

- Use `jj`, not `git`, for repository operations.
- Do not run Git write commands: `git add`, `git commit`, `git checkout`, `git switch`, `git reset`, `git rebase`, `git merge`, `git branch`, `git push`, `git pull`, `git stash`, or `git worktree`.
- Do not mix Git write operations with `jj` in the same session. Use `jj git ...` for Git remote interop.
- Use `jj --no-pager ...` for commands that may page or produce long output.
- Always pass `-m`/`--message` to commands that would otherwise open an editor: `jj commit`, `jj describe`, `jj new`, `jj split`, and `jj squash` when messages may be combined.
- Avoid interactive commands unless the user explicitly requests an interactive session and the terminal supports it. Avoid `jj split` without file paths, `jj split -i`, `jj squash -i`, `jj diffedit`, `jj arrange`, and `jj resolve` without a concrete plan.
- Quote revsets in the shell, for example `jj --no-pager log -r 'mine() & ::@'`.
- Use change IDs for mutable work. Commit IDs change when history is rewritten; change IDs stay stable.
- Before mutating history, run `jj --no-pager st` and usually `jj --no-pager log -n 12`.
- After mutating history, moving files, restoring files, rebasing, or using workspaces, verify with `jj --no-pager st` and a targeted `jj --no-pager diff` or `jj --no-pager show`.
- Do not leave completed work only in the working-copy commit `@` unless the user asked for an uncommitted diff. Finish self-contained work with `jj commit -m "..."` after validation.

## Mental Model

- The working copy is a commit, shown as `@`; there is no Git-style staging area.
- `jj commit -m "message"` describes the current working-copy commit and creates a new empty working-copy commit on top. The committed change is usually `@-` immediately afterward.
- `jj new -m "message"` creates a new empty change and edits it.
- `jj describe -m "message"` changes the description of the current change without creating a new change.
- Bookmarks are named pointers for Git branch interop. They do not automatically advance like Git branches, but they follow rewrites of the commit they point to.
- Conflicts do not block commits. Resolve conflict markers directly when practical, then verify with `jj resolve --list` and `jj --no-pager st`.
- Use `jj undo`, `jj redo`, `jj --no-pager op log`, and `jj op restore` for recovery instead of Git reset/reflog workflows.
- Workspaces are jj's equivalent of separate working copies. Each workspace has its own `@`.

## Core Workflow

The daily loop is: inspect, describe or start work, edit files, validate, commit,
then create or move bookmarks only when sharing.

```bash
jj --no-pager st
jj --no-pager diff --summary
jj --no-pager diff
# run the repo's relevant validation command(s)
jj commit -m "Implement the requested change"
jj --no-pager st
jj --no-pager log -n 8
```

If `@` already contains unrelated work before starting a task, create a new change first:

```bash
jj new -m "Start requested task"
```

If `@` is empty and you are about to start work in it, describe it:

```bash
jj describe -m "Implement requested task"
```

## Common Agent Tasks

### Set up repositories and remotes

```bash
jj version
jj git clone <git-url> [destination]
jj git init [destination]
jj git init --colocate [destination]
jj git remote list
jj git remote add origin <git-url>
jj git remote set-url origin <git-url>
jj git fetch --remote origin
jj git fetch --all-remotes
```

Use `jj git clone`, `jj git fetch`, and `jj git push`; do not use raw Git clone,
fetch, pull, or push inside a jj workflow unless the user explicitly overrides the
repository policy.

### Inspect status, history, and diffs

```bash
jj --no-pager st
jj --no-pager st -- path/to/file
jj --no-pager log -n 20
jj --no-pager log -r 'mine()'
jj --no-pager diff
jj --no-pager diff --summary
jj --no-pager diff --name-only
jj --no-pager diff -r @
jj --no-pager show @
jj --no-pager show @-
jj --no-pager show <change-id> --summary
jj file list -r @
jj file show -r <change-id> -- path/to/file
```

Use `jj diff -r <change>` to inspect what one change does relative to its parent.
Use `jj diff --from A --to B` to compare two tree states.

### Commit only selected files

Use this instead of `git add file && git commit`.

```bash
jj --no-pager diff --summary
jj --no-pager diff -- path/to/file-a path/to/file-b
jj commit -m "Commit only selected files" -- path/to/file-a path/to/file-b
jj --no-pager st
jj --no-pager show @- --summary
```

The selected paths go into the committed change. Any remaining working-copy changes
move into the new `@` on top.

### Move or remove unintended file changes

Discard an unintended path from the current working-copy commit:

```bash
jj restore -- path/to/unwanted-file
jj --no-pager st
```

Move file changes from the current working copy into an existing change:

```bash
jj squash --from @ --into <target-change-id> --use-destination-message -- path/to/file
jj --no-pager show <target-change-id> --summary
jj --no-pager st
```

Remove an unintended file change from a historical commit:

```bash
jj --no-pager show <bad-change-id> --summary
jj restore --from <bad-change-id>- --into <bad-change-id> -- path/to/file
jj --no-pager show <bad-change-id> --summary
jj --no-pager st
```

For broader history edits, read [history.md](history.md).

### Handle ignored or accidental files

Jujutsu automatically tracks new files unless they are ignored or configured
otherwise. Ignore accidental local files before untracking them.

```bash
# First add the path or pattern to .gitignore or the repo's ignore policy.
printf '\nlocal-only.log\n' >> .gitignore

# Then untrack ignored files that should not be in jj history.
jj file untrack -- local-only.log
jj --no-pager st
```

Only use `jj file untrack` for paths that are already ignored. To intentionally
track an ignored file, use:

```bash
jj file track --include-ignored -- path/to/file
```

### Share work with bookmarks and remotes

Bookmarks do not auto-advance. Set or move a bookmark before pushing.

```bash
jj bookmark list
jj bookmark create feature-name -r @
jj bookmark set feature-name -r @
jj git push --bookmark feature-name
```

After `jj commit`, the completed change is usually `@-`, not the new empty `@`:

```bash
jj git push --change @-
```

Fetch and rebase instead of using `git pull`:

```bash
jj git fetch --remote origin
jj rebase -b @ -o main
```

For PR, fork, stacked PR, and multi-remote workflows, read [sharing.md](sharing.md).

### Use workspaces for parallel work

Use `jj workspace` instead of `git worktree`.

```bash
jj workspace list
jj workspace add ../repo-task --name task -r main -m "task workspace"
cd ../repo-task
jj --no-pager st
jj --no-pager log -n 12
```

Before deleting a workspace directory, run `jj workspace forget <workspace-name>`
from another workspace or from the workspace itself.

## Git-to-JJ Translation

| Do not use Git | Use `jj` instead |
| --- | --- |
| `git status` | `jj --no-pager st` |
| `git diff` | `jj --no-pager diff` |
| `git log` | `jj --no-pager log` |
| `git add file && git commit -m` | `jj commit -m "..." -- file` |
| `git commit -m` | `jj commit -m "..."` |
| `git commit --amend` | `jj squash --into @- --use-destination-message` or `jj describe -m "..." @-` |
| `git checkout -b feature` | `jj new main -m "feature"`; optionally `jj bookmark create feature -r @` |
| `git switch branch` | `jj new <bookmark> -m "work"` or `jj edit <change-id>` when intentionally editing existing work |
| `git fetch` | `jj git fetch --remote origin` |
| `git pull --rebase` | `jj git fetch --remote origin` then `jj rebase -b @ -o main` |
| `git push` | `jj git push --bookmark <bookmark>` or `jj git push --change @-` |
| `git rebase` | `jj rebase -b/-s/-r ... -o ...` |
| `git cherry-pick` | `jj duplicate <change-id>` or `jj rebase -r <change-id> -o <dest>` |
| `git reset --hard` | `jj restore`, `jj abandon`, `jj undo`, or `jj op restore` depending on intent |
| `git stash` | `jj new -m "scratch"`, `jj squash`, or a separate `jj workspace` |
| `git worktree` | `jj workspace` |
| `git branch` | `jj bookmark` |

## Common Mistakes

| Mistake | Fix |
| --- | --- |
| Omitting `--no-pager` | Always pass `--no-pager` on log, diff, show, op log, and other possibly long output |
| Omitting `-m` on commands | Always pass `-m` so jj does not open an editor |
| Using interactive split/squash/resolve | Use file-path arguments or direct conflict-marker edits |
| Forgetting to move a bookmark before push | `jj bookmark set <name> -r @` or `jj bookmark set <name> -r @-` first |
| Pushing after `jj commit` with `@` | Push `@-` or move the bookmark to `@-` |
| Using commit IDs for mutable work | Prefer change IDs, which survive rewrites |
| Running Git write commands | Use jj equivalents or `jj git ...` for remote interop |

## Recovery

```bash
jj undo
jj redo
jj --no-pager op log -n 10
jj --no-pager op show <operation-id>
jj op restore <operation-id>
jj --no-pager evolog -r @
jj --no-pager evolog -p -r <change-id>
```

Use `jj undo` first for a mistaken local operation. Use `jj op restore <operation-id>`
only when you intentionally want to restore repository state to that operation.

## Additional Resources

- [Official jj documentation](https://docs.jj-vcs.dev/latest/)
- [Steve Klabnik's Jujutsu Tutorial](https://github.com/steveklabnik/jujutsu-tutorial)
- [jj GitHub repository](https://github.com/jj-vcs/jj)
