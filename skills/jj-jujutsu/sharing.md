# Sharing and Collaboration

Bookmarks, remotes, pushing, pulling, and PR workflows.

---

## Bookmarks (jj's Branches)

Bookmarks are named pointers to commits — jj's equivalent of Git branches.

**Bookmarks do NOT auto-advance.** When you create a new commit with `jj new`, `@`
moves but bookmarks stay where they are.

**Bookmarks DO follow rewrites.** `jj rebase`, `jj squash`, `jj abandon` — if these
modify the commit a bookmark points at, the bookmark moves to the rewritten version.

```bash
# Create a new bookmark on current commit
jj bookmark create feat-auth -r @

# Move an existing bookmark to current commit
jj bookmark set feat-auth -r @

# Delete a bookmark
jj bookmark delete feat-auth

# List all bookmarks (shows tracking status)
jj --no-pager bookmark list

# Track a remote bookmark
jj bookmark track main@origin
```

---

## Pushing Changes

The canonical push pattern: **describe → set bookmark → push.**

### Standard Push

```bash
# Describe your work
jj describe -m "feat: add auth middleware"

# Set the bookmark
jj bookmark set feat-auth -r @

# Push
jj git push -b feat-auth
```

### Auto-Named Push (one-off PR)

```bash
# Push current commit with auto-generated bookmark name
jj git push -c @
```

### After `jj commit`

`jj commit` finalizes `@` and creates an empty child. Your work is now in `@-`:

```bash
jj commit -m "feat: done with auth"
# Work is at @-, not @
jj git push -c @-
```

### Push Safety

Push is safe by default (like `--force-with-lease`). It rejects if the remote
changed since your last fetch:

```bash
jj git fetch
jj git push -b feat-auth
```

---

## Fetching Changes

There is no `jj git pull`. The idiom is **fetch + rebase:**

```bash
# Fetch all remotes
jj git fetch

# Rebase your work onto updated trunk
jj rebase -s 'all:roots(trunk()..mine())' -o trunk()
```

---

## Feature Branch / PR Workflow

### Creating a PR

```bash
# 1. Do your work
jj new -m "feat: add user search"
# ... code ...

# 2. Bookmark the commit
jj bookmark create feat-user-search -r @

# 3. Push
jj git push -b feat-user-search

# 4. Open PR (via web UI, gh cli, etc.)
```

### Updating a PR via New Commits

```bash
# Add more work on top
jj new -m "feat: add search filters"
# ... code ...

# Move the bookmark forward
jj bookmark set feat-user-search -r @

# Push (jj handles force-push automatically)
jj git push -b feat-user-search
```

### Updating a PR via Rewriting (Amend in Place)

```bash
# Edit the original commit directly
jj edit <change-id>

# Make changes (auto-snapshotted)

# Push — jj force-pushes the rewritten commit
jj git push -b feat-user-search
```

---

## Stacked PRs

```bash
# Build the stack
jj new trunk() -m "refactor: extract auth module"
jj bookmark create pr-1 -r @
# ... code ...

jj new -m "feat: add OAuth support"
jj bookmark create pr-2 -r @
# ... code ...

# Push all at once
jj git push -b pr-1 -b pr-2
```

---

## Independent Parallel PRs

```bash
# Start two independent lines from trunk
jj new trunk() -m "fix: timezone handling"
jj bookmark create fix-tz -r @
# ... fix ...

jj new trunk() -m "feat: add dark mode"
jj bookmark create feat-dark -r @
# ... code ...

# Push independently
jj git push -b fix-tz -b feat-dark
```

---

## Working with Multiple Remotes (Fork Workflow)

```bash
# Add upstream remote
jj git remote add upstream https://github.com/original/repo.git

# Fetch from upstream
jj git fetch --remote upstream

# Rebase onto upstream's main
jj rebase -s 'all:roots(trunk()..mine())' -o trunk()

# Push to your fork
jj git push -b my-feature
```

---

## Using GitHub CLI

In colocated repos (`.jj/` + `.git/`), `gh` works normally.

In non-colocated repos, set the git dir:

```bash
GIT_DIR=.jj/repo/store/git gh pr create
```

---

## Agent Checklist (Before Any Push)

1. Verify commit has a description: `jj --no-pager log -r @`
2. Ensure a bookmark points at it: `jj --no-pager bookmark list`
3. If not, create/set one: `jj bookmark set <name> -r @`
4. Check for conflicts: `jj --no-pager st`
5. Push: `jj git push -b <name>`
6. Verify: `jj --no-pager bookmark list`

---

## Common Mistakes

| Mistake                                 | Fix                                          |
|-----------------------------------------|----------------------------------------------|
| Forget to move bookmark before push     | `jj bookmark set <name> -r @` first         |
| Pushing empty working copy after commit | Push `@-` not `@`                            |
| Not fetching before push                | `jj git fetch` first                         |
| Running `git push` in colocated repo    | Use `jj git push` instead                    |
| Expecting bookmarks to auto-advance     | They don't — set explicitly before pushing   |
