# Git Interop

How jj relates to Git — the colocated repo model, concept mapping, when to use
raw `git`, and safety at the boundary.

---

## The Big Picture

jj uses Git as its storage backend. Every jj repo has a Git repo inside it —
either visible (colocated) or hidden. Your commits are real Git commits. Your
remotes are real Git remotes. Collaborators don't need to know you're using jj.

---

## Colocated Repos

A colocated repo has both `.jj/` and `.git/` in the same directory. This is the
default and the common case.

**"Detached HEAD" is normal.** Git will report detached HEAD in colocated repos.
This is expected — jj has no concept of a "current branch." Use
`jj --no-pager log` for the real state, not `git log`.

### Why Colocated Matters

- Build tools, CI, and IDEs that expect `.git/` work normally
- You can mix `jj` and `git` read commands (with care)
- `jj` auto-syncs with `.git/` on every command

---

## Concept Mapping

| Git concept               | jj equivalent                | Key difference                                      |
|---------------------------|------------------------------|-----------------------------------------------------|
| Staging area (`git add`)  | No equivalent                | Working copy auto-commits; use `jj split`/`squash`  |
| Branches                  | Bookmarks                    | Don't auto-advance; must be set explicitly           |
| `HEAD` / current branch   | `@` (working copy commit)    | Always exists, always points to a real commit        |
| Detached HEAD             | Normal state                 | jj doesn't track a "current bookmark"                |
| Reflog                    | Operation log (`jj op log`)  | Tracks entire repo state, not per-ref                |
| `git stash`               | `jj new @-`                  | Old working copy stays as a sibling commit           |
| `git commit --amend`      | `jj squash` or `jj describe` | Descendants auto-rebase                             |
| `git rebase -i`           | `jj rebase -r`, `jj squash`  | No interactive mode needed; each op is atomic        |
| `git cherry-pick`         | `jj duplicate`               |                                                      |
| `git revert`              | `jj revert`                  |                                                      |
| `git worktree`            | `jj workspace`               | Native support, no Git worktrees involved            |
| Merge conflicts block     | Conflicts can be committed   | Resolve later; conflicts are data, not errors        |

---

## When to Use Raw `git`

jj handles most daily work. Use raw `git` only for features jj doesn't support:

| Feature                | jj support      | What to do                         |
|------------------------|-----------------|-------------------------------------|
| Submodules             | No              | Use `git submodule` commands        |
| Git LFS               | No              | Use `git lfs` commands              |
| Annotated tags         | No (lightweight) | Use `git tag -a`                   |
| `.gitattributes`       | No              | Edit the file directly              |
| Pre-commit hooks       | No              | Run hook tools directly             |
| Partial/shallow clones | Limited         | Use `git clone --depth` then `jj git init` |

**After any mutating `git` command in a colocated repo, run any `jj` command**
(even `jj --no-pager st`) to re-sync state.

---

## Mixing Commands Safely

**Safe to mix:**
- Read-only `git` commands (`git log`, `git show`, `git diff`, `git blame`)
- `git fetch` — safe, but prefer `jj git fetch` (auto-tracks bookmarks)

**Use with care:**
- `git commit`, `git rebase`, `git merge` — can cause divergent change IDs
- `git switch` / `git checkout` — may be needed before mutating git commands

**Avoid:**
- `git push` — use `jj git push`; raw push desyncs bookmark tracking
- `git reset` — use `jj abandon`, `jj restore`, or `jj op restore`
- `git add` — not needed; jj auto-tracks

### Recovery from Mixed-Command Issues

```bash
jj --no-pager op log            # see what happened
jj undo                          # undo the last jj operation
jj op restore <op-id>           # restore to a known-good state
```

---

## Agent Rules at the Git Boundary

1. **Always prefer `jj` commands** over `git` equivalents in jj repos
2. **Use `jj git push`**, never raw `git push`
3. **After raw `git` mutations**, run `jj --no-pager st` to re-sync
4. **Don't panic at "detached HEAD"** — it's normal in colocated repos
5. **For unsupported features** (submodules, LFS, hooks), use `git` directly
