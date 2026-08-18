# Jujutsu-First Local Workflow

This reference defines how to choose between Jujutsu and Git and how to keep
Forgejo operations separate from local version-control operations.

## Select the VCS Deliberately

Use Jujutsu only when it is both installed and active for the current
repository:

```bash
if command -v jj >/dev/null 2>&1 && jj root >/dev/null 2>&1; then
  printf '%s\n' "Use Jujutsu"
elif git rev-parse --show-toplevel >/dev/null 2>&1; then
  printf '%s\n' "Use Git"
else
  printf '%s\n' "No local repository detected"
fi
```

Do not equate "`jj` exists on PATH" with "this existing repository uses
`jj`". Do not run `jj git init`, `jj git init --colocate`, or any conversion
command against an existing checkout without an explicit request.

Once a mode is selected, stay in that mode for the task. Do not mix Git writes
into a Jujutsu session.

## New Clone Exception

A clone has no existing repository state to detect. When cloning into a new
destination, prefer Jujutsu whenever `jj` is installed unless the user asks for
a Git-managed checkout:

```bash
if command -v jj >/dev/null 2>&1; then
  jj git clone <forgejo-repository-url> [destination]
else
  fj repo clone owner/repo [destination]
fi
```

After cloning with `jj`, verify with `jj root`, `jj --no-pager st`, and
`jj git remote list`. Do not use this exception to convert a directory that
already contains a Git checkout.

## Jujutsu Mode

### Read-only inspection

```bash
jj --no-pager st
jj --no-pager diff --summary
jj --no-pager diff
jj --no-pager log -n 20
jj --no-pager log -r 'mine()'
jj git remote list
jj bookmark list
```

Use `--no-pager` for output that can page. Quote revsets.

### Start or describe work

If `@` already contains unrelated work:

```bash
jj new -m "Start requested Forgejo task"
```

If `@` is empty and will hold the new work:

```bash
jj describe -m "Implement requested Forgejo task"
```

Always pass a message to commands that would otherwise open an editor.

### Commit completed work

```bash
jj --no-pager diff --summary
jj --no-pager diff
# run the repository's relevant validation
jj commit -m "Implement the requested change"
jj --no-pager st
jj --no-pager show @- --summary
```

After `jj commit`, the completed change is normally `@-`; `@` is the new empty
working-copy commit.

### Fetch

```bash
jj git fetch --remote origin
```

Do not use `git fetch`, `git pull`, or `fj` commands that fetch and check out
through Git when Jujutsu mode is active.

Rebase explicitly when the task requires it:

```bash
jj rebase -b @ -o main
```

Use the repository's actual trunk bookmark rather than assuming `main`.

### Create or move a bookmark

Before sharing a completed change:

```bash
jj bookmark create feature-name -r @-
```

If the bookmark already exists:

```bash
jj bookmark set feature-name -r @-
```

Inspect before moving a shared bookmark:

```bash
jj bookmark list
jj --no-pager show @- --summary
```

### Push

```bash
jj git push --bookmark feature-name
```

For a one-off change when a named bookmark is not required:

```bash
jj git push --change @-
```

A generated bookmark may be created by `--change`. Inspect the result before
creating a pull request.

Do not use raw `git push` in Jujutsu mode.

### Workspaces

Use Jujutsu workspaces instead of Git worktrees:

```bash
jj workspace list
jj workspace add ../repo-task --name repo-task -r main \
  -m "Work on Forgejo task"
```

Do not use `git worktree` in Jujutsu mode.

## Forgejo Commands That Can Cross the Boundary

Most `fj` commands only manipulate Forgejo objects and are safe to use beside
Jujutsu. Avoid commands that manipulate a local checkout through Git when
Jujutsu mode is active.

| Avoid in Jujutsu mode | Use instead |
| --- | --- |
| `fj repo clone ...` | Obtain the repository URL and use `jj git clone ...` |
| `fj pr checkout ...` | Inspect with `fj pr view`; fetch the head with `jj git fetch`; create/edit a jj change deliberately |
| `fj repo create ... --push` | Create remotely without automatic push, add the remote with `jj git remote add`, then `jj git push` |
| Raw Git commit, rebase, branch, push, pull, stash, or worktree commands | Jujutsu equivalents |

`fj pr checkout` may be used only after the task has selected Git mode.

## Inspect a Pull Request Without Checking It Out

For review or diagnosis, prefer remote inspection:

```bash
fj pr view owner/repo#45
fj pr view owner/repo#45 files
fj pr view owner/repo#45 commits --oneline
fj pr view owner/repo#45 diff
fj pr view owner/repo#45 comments
```

This avoids mutating the local working copy.

If a local checkout of the pull request is genuinely needed in Jujutsu mode:

1. inspect the pull request to identify its source repository and source
   branch;
2. inspect existing remotes with `jj git remote list`;
3. add a remote with `jj git remote add` only when necessary;
4. fetch it with `jj git fetch --remote <remote>`;
5. inspect imported bookmarks and commits;
6. start a new change from the intended commit with `jj new <revision> -m ...`
   or deliberately edit it with `jj edit <change-id>`.

Do not guess a Forgejo pull-request ref namespace. Use the source branch or
confirm the server-specific ref from installed help or repository policy.

## Repository Target Detection

`fj` and `fj-ex` commonly infer a target from Git remotes. A colocated Jujutsu
repository normally exposes Git metadata, but a non-colocated repository may
not provide the context those tools expect.

When inference fails or multiple remotes exist:

```bash
fj --host forge.example.com pr view owner/repo#45

fj-ex actions runs \
  --host forge.example.com \
  --repo owner/repo \
  --limit 20
```

Prefer explicit host and repository over creating temporary Git state solely
for CLI inference.

## Git Fallback Mode

Use Git only when the current repository is not a Jujutsu repository.

Read-only inspection:

```bash
git status --short --branch
git diff --stat
git diff
git log --oneline -n 20
git remote -v
```

For writes, follow the repository's existing branch and commit policy. Inspect
before committing and pushing. Never use a force push unless explicitly
required and the target has been verified.

If `jj` becomes available after the task starts, do not switch modes midway.
Complete the task consistently or stop and deliberately restart the workflow.
