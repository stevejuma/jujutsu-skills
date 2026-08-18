# Forgejo Pull Requests with `fj`

Use stock `fj` for pull-request discovery, inspection, creation, comments,
status, review information, and merge operations.

## Search and Inspect

```bash
fj pr search --repo owner/repo --state open
fj pr view owner/repo#45
fj pr view owner/repo#45 body
fj pr view owner/repo#45 files
fj pr view owner/repo#45 commits --oneline
fj pr view owner/repo#45 diff
fj pr view owner/repo#45 comments
fj pr review owner/repo#45 list
```

For a code review, inspect at least:

- title and body;
- base and head;
- changed files;
- full diff;
- commits;
- comments and review state;
- current check status.

Do not rely only on the PR description.

## Jujutsu-First Sharing

When the current repository is a Jujutsu repository, use Jujutsu for local
work and sharing:

```bash
jj --no-pager st
jj --no-pager diff
# run validation
jj commit -m "Implement requested change"
jj bookmark create feature-name -r @-
jj git push --bookmark feature-name
```

If the bookmark already exists:

```bash
jj bookmark set feature-name -r @-
jj git push --bookmark feature-name
```

Then create or update the Forgejo pull request with `fj`.

Do not use `git push`, `git branch`, or `fj pr checkout` in Jujutsu mode. Read
[jj-first.md](jj-first.md) for pull-request inspection and checkout guidance.

## Create a Pull Request

Inspect the command for the installed version:

```bash
fj pr create --help
```

Typical stock `fj` usage:

```bash
fj pr create \
  --repo owner/repo \
  --base main \
  --head feature-name \
  "Concise pull-request title" \
  --body-file /tmp/forgejo-pr-body.md
```

Use `--autofill` only when commit descriptions contain an accurate title and
body.

`fj` also supports AGit pull requests. Use `--agit` only when the repository
and user workflow intentionally use AGit; do not switch an ordinary
fork/branch workflow to AGit without instruction.

Before creation:

1. confirm the pushed bookmark or branch exists remotely;
2. confirm the base branch;
3. review the exact diff;
4. run relevant validation;
5. avoid unrelated changes;
6. include testing and risk information.

## Comment

```bash
fj pr comment owner/repo#45 \
  --body-file /tmp/forgejo-pr-comment.md
```

Read existing comments first. Do not duplicate an existing status update or
post raw sensitive logs.

## Monitor Status

Stock `fj` can report PR status and wait:

```bash
fj pr status owner/repo#45
fj pr status owner/repo#45 --wait
```

Also inspect Forgejo Actions:

```bash
fj actions --repo owner/repo tasks
```

For job-level monitoring, logs, and artifacts, use `fj-ex` as described in
[actions.md](actions.md).

A green status does not replace code review. A successful Actions run does
not by itself authorize a merge.

## Review

List existing reviews:

```bash
fj pr review owner/repo#45 list
```

Before submitting any review mutation supported by the installed CLI:

- inspect the full diff;
- distinguish blocking defects from suggestions;
- cite files and lines where possible;
- do not approve code you did not meaningfully review;
- do not dismiss another person's review without authority.

Run `fj pr review --help` because review-writing capabilities may differ by
version.

## Close

```bash
fj pr close owner/repo#45
```

Close only when requested or when the task explicitly establishes that the PR
is obsolete. Explain why in a comment when repository policy expects it.

## Merge

Inspect the PR immediately before merge:

```bash
fj pr view owner/repo#45
fj pr status owner/repo#45
fj pr review owner/repo#45 list
```

Then, only when authorized:

```bash
fj pr merge owner/repo#45 --method squash
```

Stock `fj` supports merge methods including merge, rebase, rebase-merge,
squash, and manual. Use the repository's established method. Do not choose a
method merely for convenience.

`--delete` deletes the source branch after merge. Treat it as a separate
destructive choice; do not add it unless requested or clearly required by
repository policy.

Before merge, verify:

- the exact repository and PR number;
- the base branch;
- review requirements;
- Actions status for the current head;
- unresolved conversations;
- merge conflicts;
- release or deployment implications.

After merge, fetch and verify the resulting state. In Jujutsu mode:

```bash
jj git fetch --remote origin
jj --no-pager log -n 12
```

Do not claim that a local bookmark moved automatically. Jujutsu bookmarks do
not behave exactly like Git branches.

## Updating an Existing Pull Request

1. Read the PR and latest comments.
2. Identify the current head change.
3. Fetch with the selected VCS.
4. Make the smallest necessary fix.
5. Validate locally.
6. Commit with Jujutsu or Git.
7. Move the Jujutsu bookmark when needed.
8. Push.
9. Confirm the PR head changed;
10. monitor the new Actions run rather than an older successful run.

When using `fj-ex --latest`, guard against races by checking the workflow and
commit SHA before diagnosing or reporting the result.
