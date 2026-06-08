# Jujutsu (jj) Version Control — Agent Skill

This repository uses [Jujutsu (jj)](https://github.com/jj-vcs/jj), a Git-compatible
VCS with mutable commits, automatic change tracking, and an operation log.

**Tested with jj 0.39.0** — Commands may differ in other versions.

## Deep-Dive Topics

| I need to...                                    | File                                   |
| ----------------------------------------------- | -------------------------------------- |
| Disable pager or understand which commands page | [pager-commands.md](pager-commands.md) |
| Understand how jj relates to Git                | [git-interop.md](git-interop.md)       |
| Write revset, fileset, or template expressions  | [revsets.md](revsets.md)               |
| Push, pull, manage bookmarks, or work with PRs  | [sharing.md](sharing.md)               |
| Split, rebase, squash, or resolve conflicts     | [history.md](history.md)               |
| Configure jj, set up aliases, or customize      | [config.md](config.md)                 |

---

## CRITICAL: Pager Will Hang Agents

jj uses a pager (`less` by default) for **all commands** when `ui.paginate = "auto"`
(the default). This **will hang** any non-interactive agent.

**Always use `--no-pager`** as a global flag, or set the config:

```bash
# Option 1: Global flag on every command (preferred for agents)
jj --no-pager log
jj --no-pager diff
jj --no-pager show @

# Option 2: Set config once per repo
jj config set --repo ui.paginate never

# Option 3: Environment variable
JJ_PAGER=cat jj log
```

See [pager-commands.md](pager-commands.md) for the full list and details.

---

## Agent Rules (Non-Negotiable)

1. **Always use `--no-pager`** on every `jj` command, or configure `ui.paginate = "never"`.
   Agents get stuck waiting for pager input.

2. **Always use `-m` for messages.** Never invoke a command that opens an editor.
   Commands that need `-m`: `jj describe`, `jj commit`, `jj squash`, `jj new`.

3. **Never use interactive commands.** `jj split` (without file paths), `jj squash -i`,
   `jj resolve`, `jj arrange`, `jj diffedit` — all hang. Use file-path args or
   `jj restore` workflows instead.

4. **Verify after mutations.** Run `jj --no-pager st` after `squash`, `abandon`,
   `rebase`, `restore`, or any destructive operation.

5. **Use change IDs, not commit IDs.** Change IDs (letters k–z, e.g. `tqpwlqmp`) survive
   rewrites. Commit IDs (hex digits) change on any rewrite.

6. **Quote revsets.** Always single-quote revset expressions in shell:
   `jj --no-pager log -r 'mine() & ::@'`.

7. **Do NOT run `git commit`, `git add`, or other git write commands.** In a jj repo,
   git write operations can corrupt state. Use `jj` equivalents.

8. **Do not mix `jj` and `git` write operations** in the same session.

---

## Mental Model

**The working copy IS a commit.** No staging area. Every file change is
auto-snapshotted into `@` when you run any `jj` command. Instead of
"stage → commit," just code and describe.

**Change IDs are stable.** Commit IDs are not.

**History is mutable.** Commits can be freely rewritten. Descendants auto-rebase.
Old versions stay in the operation log (`jj op log`).

**Bookmarks are NOT branches.** Bookmarks don't advance when new commits are created.
They follow rewrites but must be explicitly set before pushing.

**Conflicts don't block.** jj allows committing conflicted files. Resolve at your
convenience by editing conflict markers directly, then verify with `jj --no-pager st`.

## Core Workflow

The daily loop: **describe → code → new → repeat.**

```bash
# 1. Check if working copy already has changes
jj --no-pager st

# 2. If @ already has changes, start a new commit first
jj new -m "feat: add user validation"

# 3. If @ is empty, describe it
jj describe -m "feat: add user validation"

# 4. Make your changes — auto-tracked, no `add` needed

# 5. Verify
jj --no-pager st && jj --no-pager diff

# 6. Start next task (creates new empty commit on top)
jj new -m "feat: add error handling"
```

### Committing (This Repo's Convention)

This repository uses `jj commit` to finalize changes:

```bash
# After verified, self-contained change (compiles, tests pass, lints pass):
jj commit -m "<descriptive message>"
```

Commit messages: imperative mood, ≤ 72 characters first line, optional body
separated by a blank line.

### Curating History

```bash
jj squash -m "feat: final clean message"   # fold working copy into parent
jj absorb                                   # auto-distribute hunks to right ancestor
jj abandon @                               # drop a failed experiment
```

→ Deep dive: [history.md](history.md)

### Pushing Changes

```bash
jj bookmark set feat -r @
jj git push -b feat
```

Bookmarks must be set before pushing — they don't auto-advance.
→ Deep dive: [sharing.md](sharing.md)

### Essential Commands (Quick Reference)

| Task                     | Command                               |
| ------------------------ | ------------------------------------- |
| Check status             | `jj --no-pager st`                    |
| View diff                | `jj --no-pager diff`                  |
| View log                 | `jj --no-pager log`                   |
| Describe current commit  | `jj describe -m "message"`            |
| Commit (finalize)        | `jj commit -m "message"`              |
| Start new work           | `jj new -m "task description"`        |
| Edit an older commit     | `jj edit <change-id>`                 |
| Show a commit            | `jj --no-pager show <change-id>`      |
| Squash into parent       | `jj squash -m "message"`              |
| Auto-distribute changes  | `jj absorb`                           |
| Abandon a commit         | `jj abandon <change-id>`              |
| Undo last operation      | `jj undo`                             |
| View operation history   | `jj --no-pager op log`                |
| Restore to earlier state | `jj op restore <op-id>`               |
| Restore files            | `jj restore [paths]`                  |
| Create bookmark          | `jj bookmark create <name> -r @`      |
| Set/move bookmark        | `jj bookmark set <name> -r @`         |
| Push bookmark            | `jj git push -b <name>`               |
| Fetch from remote        | `jj git fetch`                        |
| View commit evolution    | `jj --no-pager evolog -r <change-id>` |

### Recovery

```bash
jj undo                          # undo last op; repeatable
jj --no-pager op log             # full operation history
jj op restore <op-id>            # jump to any past state
jj --no-pager evolog -r <id>     # see how a change evolved
```

### Detecting a jj Repo

- `.jj/` directory = jj repo.
- Both `.jj/` and `.git/` = colocated repo (this is the common case).
- Always use `jj` commands. Git's "detached HEAD" is normal in colocated repos — use
  `jj --no-pager log` for real state.

### Common Mistakes

| Mistake                                | Fix                                                         |
| -------------------------------------- | ----------------------------------------------------------- |
| Omitting `--no-pager`                  | Always pass `--no-pager` — pager hangs agents               |
| Omitting `-m` on commands              | Always pass `-m` — editor hangs agents                      |
| Using `jj split` without file paths      | Provide paths or use the `jj restore` workflow              |
| Forgetting bookmark before push          | `jj bookmark set <name> -r @` first                         |
| Using commit IDs instead of change IDs    | Change IDs (letters k–z) survive rewrites                   |
| Unquoted revset expressions             | Always single-quote: `'mine() & ::@'`                       |
| Running `git commit`/`git add`           | Use `jj commit`/auto-tracking instead                       |
| Running `git push`                        | Use `jj git push` instead                                   |
| Using `jj squash -i`                     | Interactive — hangs. Use `jj squash` or `jj squash <paths>` |
| Using `jj resolve`                        | Interactive — hangs. Edit conflict markers directly         |

### Additional Resources

- [Official jj documentation](https://docs.jj-vcs.dev/latest/)
- [Steve Klabnik's Jujutsu Tutorial](https://github.com/steveklabnik/jujutsu-tutorial)
- [jj GitHub repository](https://github.com/jj-vcs/jj)
