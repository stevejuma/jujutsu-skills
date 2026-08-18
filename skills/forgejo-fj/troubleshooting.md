# Troubleshooting and Version Drift

Use this reference when commands, authentication, repository detection, or
Actions behaviour do not match expectations.

## Wrong `fj` Command Detected

Multiple unrelated tools use the executable name `fj`.

Inspect:

```bash
command -v fj
fj version
fj --help
```

This skill expects the `forgejo-contrib/forgejo-cli` command tree with groups
such as:

```text
repo  issue  pr  wiki  actions  release  tag  user  org  auth  completion
```

A different `fj` may expose groups such as `run`, `workflow`, `secret`, and
`variable` directly. Do not apply this skill's command syntax to that CLI.

Resolution:

1. identify the installed package and version;
2. use its installed help as the source of truth;
3. do not uninstall or replace it without instruction;
4. when the user intended `forgejo-contrib/forgejo-cli`, install or select that
   binary through the user's normal package-management process;
5. avoid aliasing two different tools to the same executable in an agent
   environment.

## Command or Flag Not Recognized

```bash
fj <group> --help
fj <group> <subcommand> --help
```

For the extension:

```bash
fj-ex <group> --help
fj-ex <group> <subcommand> --help
```

Do not retry with guessed flags. Note the installed version and adapt to the
documented command surface.

## Repository Not Detected

Symptoms include missing repository errors, a command targeting the wrong
remote, or `fj-ex` failing to infer the host.

In Jujutsu mode:

```bash
jj root
jj git remote list
```

In Git mode:

```bash
git rev-parse --show-toplevel
git remote -v
```

Then pass explicit context:

```bash
fj --host forge.example.com issue view owner/repo#123

fj-ex actions runs \
  --host forge.example.com \
  --repo owner/repo \
  --limit 20
```

Do not initialize Git inside a non-colocated Jujutsu repository solely to make
automatic detection work.

## Wrong Remote or Fork Selected

Before a mutation:

```bash
jj git remote list
# or, in Git mode:
git remote -v
```

Inspect the Forgejo object with an explicit owner/repository. Distinguish
upstream from fork. Do not assume `origin` is upstream.

For pull requests, verify base repository, base branch, head repository, and
head branch.

## Authentication Failure

```bash
fj auth list
```

Then confirm:

- host;
- authenticated account;
- token validity;
- repository membership;
- operation-specific permissions;
- whether organization or admin scope is genuinely required.

Do not print token files or broaden permissions blindly.

For `fj-ex`:

```bash
fj-ex auth status --host forge.example.com
```

Do not repeatedly retry a bad password or one-time-password flow.

## `fj` Shows Actions but Cannot Retrieve Logs

First feature-detect the installed build:

```bash
fj actions --help
```

The upstream `forgejo-contrib` 0.6.0 baseline provides task overview,
dispatch, variables, and secrets. Some downstream builds add `actions jobs`
and `actions logs` without changing the base version.

When those commands are present:

```bash
fj actions --repo owner/repo jobs <run-id>
fj actions --repo owner/repo logs --job <job-id>
```

If they are absent or unsupported by the server, check for the extension:

```bash
command -v fj-ex
```

When available, use:

```bash
fj-ex actions jobs --repo owner/repo --latest --watch
fj-ex actions logs job --repo owner/repo --latest --job-index 0
```

When unavailable, report the missing capability rather than claiming that no
logs exist.

## `--latest` Selected the Wrong Run

A newer workflow may start between listing and diagnosis.

Mitigation:

1. filter by workflow;
2. record the run index and commit;
3. use the explicit run index for logs, artifacts, rerun, or cancel;
4. re-check immediately before mutation.

Never cancel or rerun solely on the basis of an unverified `--latest`.

## Actions Run Is Stuck

Inspect:

```bash
fj actions --repo owner/repo tasks
fj-ex actions jobs --repo owner/repo --latest
fj-ex actions runners jobs --repo owner/repo --waiting
```

Possible causes include unavailable runner labels, offline runners, capacity,
concurrency, approval gates, or a job waiting on an external service.

Do not cancel until the target run and deployment implications are understood.

## Jujutsu and Git State Diverged

In Jujutsu mode:

```bash
jj --no-pager st
jj --no-pager log -n 20
jj --no-pager op log -n 10
jj git remote list
```

Do not repair with `git reset`, `git checkout`, or `git rebase`.

Use Jujutsu recovery:

```bash
jj undo
jj redo
jj op restore <operation-id>
```

Read the companion `jj-jujutsu` skill for detailed recovery and Git interop.

## Pull Request Did Not Update After Push

In Jujutsu mode, verify the completed change and bookmark:

```bash
jj --no-pager show @- --summary
jj bookmark list
jj git push --bookmark feature-name
```

A Jujutsu bookmark does not automatically advance like a Git branch. Move it
to the intended change before pushing:

```bash
jj bookmark set feature-name -r @-
```

Then re-open the PR and verify its head.

## Secret or Credential Appeared in Output

Stop propagating the value.

1. redact it from agent output and temporary files;
2. do not commit logs or transcripts containing it;
3. rotate the credential when exposure is plausible;
4. remove public comments or artifacts only with appropriate authority;
5. report the exposure without repeating the secret.

Do not hide a confirmed credential exposure from the user.

## Temporary Logs or Artifacts

Store downloaded data under an ignored temporary directory such as:

```text
.tmp/forgejo-logs/
.tmp/forgejo-artifacts/
```

Check ignore rules before download. Remove temporary sensitive material after
the task when doing so does not destroy evidence the user asked to retain.

## Reporting an Unresolved Failure

Report:

- `fj`, `fj-ex`, and `jj` versions when relevant;
- selected host and repository;
- selected VCS mode;
- exact non-secret command;
- concise error;
- read-only checks already performed;
- what remains unavailable or ambiguous.

Never report a mutation as successful unless the resulting Forgejo state was
verified.
