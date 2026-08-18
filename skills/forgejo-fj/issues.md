# Forgejo Issues with `fj`

Use stock `fj` for issue operations. Read the issue and its recent discussion
before modifying it.

## Identify the Target

Prefer an explicit reference:

```bash
fj --host forge.example.com issue view owner/repo#123
```

A bare number is acceptable only when the current repository and host are
unambiguous.

## Search

```bash
fj --host forge.example.com issue search \
  --repo owner/repo \
  --state open
```

Useful filters include query text, labels, creator, assignee, and state. Check
the installed syntax before combining unfamiliar filters:

```bash
fj issue search --help
```

Examples:

```bash
fj issue search --repo owner/repo "authentication"
fj issue search --repo owner/repo --labels bug --state open
fj issue search --repo owner/repo --assignee username --state all
```

## Inspect

```bash
fj issue view owner/repo#123
fj issue view owner/repo#123 body
fj issue view owner/repo#123 comments
fj issue view owner/repo#123 assignees
```

Inspect the body and comments before deciding that an issue is stale,
duplicated, resolved, or safe to close.

## Create

Prefer a body file for non-trivial content:

```bash
fj issue create \
  --repo owner/repo \
  "Concise issue title" \
  --body-file /tmp/forgejo-issue-body.md
```

Before creating:

1. search for duplicates;
2. confirm the target repository;
3. include reproduction steps, expected behaviour, actual behaviour, and
   acceptance criteria when relevant;
4. avoid secrets and private log data.

Use `--web` only when the user requests an interactive browser workflow.

## Comment

```bash
fj issue comment owner/repo#123 \
  --body-file /tmp/forgejo-issue-comment.md
```

Read the current issue immediately before posting. Do not post speculative
status as fact. Include relevant commit, pull-request, or Actions references
when available.

## Assign and Unassign

```bash
fj issue assign owner/repo#123 username
fj issue unassign owner/repo#123 username
```

Do not assign another person unless the task or repository policy authorizes
it.

## Edit

Inspect installed edit syntax before modifying an issue:

```bash
fj issue edit --help
```

For example, stock `fj` supports editing issue title, body, comments, and
labels. Read the current value first and make the smallest necessary change.

Do not replace a complete issue body merely to add a small note when a comment
would preserve history more clearly.

## Close

```bash
fj issue close owner/repo#123
```

To close with a message, consult the installed command:

```bash
fj issue close --help
```

Before closing:

- confirm the requested outcome is complete;
- link the implementing pull request or change;
- confirm any required Actions passed;
- check whether the issue has unresolved acceptance criteria;
- avoid closing merely because the latest comment is old.

Closing is a mutation. Do it only when requested or when the user's task
unambiguously includes completing and closing the issue.

## Default Issue-to-Change Workflow

1. Search for the issue and duplicates.
2. Read its body, comments, assignees, and labels.
3. Detect Jujutsu or Git using [jj-first.md](jj-first.md).
4. Make and validate the local change.
5. Commit and push with the selected VCS.
6. Create or update the pull request with `fj pr`.
7. Monitor Actions.
8. Comment on the issue with the concrete outcome when requested.
9. Close only after the completion criteria are satisfied.

## Batch Triage

For multiple issues:

- start with read-only search;
- group by a clear criterion;
- show the proposed mutations before applying them;
- avoid mass closing, reassignment, or relabelling without explicit approval;
- process in bounded batches and verify results.

Do not treat search output as a complete substitute for reading each issue
that will be mutated.
