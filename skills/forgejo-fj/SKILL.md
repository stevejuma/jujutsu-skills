---
name: forgejo-fj
description: Use Forgejo through the forgejo-contrib `fj` CLI for repositories, issues, pull requests, releases, and Actions. Prefer Jujutsu (`jj`) for local version-control work when the current repository is a jj repository, fall back to Git otherwise, feature-detect job and log support built into `fj`, and use `fj-ex` only for Actions capabilities the installed `fj` lacks.
license: MIT
compatibility: Requires the forgejo-contrib `fj` CLI. Optionally uses `jj` when installed in a Jujutsu repository and `fj-ex` for Actions capabilities unavailable in the installed fj build. Based on upstream fj 0.6.0 and fj-ex 0.1.12 with runtime feature detection; installed command help is authoritative.
metadata:
  version: "0.1.0"
  source: "Created for stevejuma/jj-jujutsu-skill as a Jujutsu-first Forgejo companion skill."
allowed-tools: Bash(fj:*), Bash(fj-ex:*), Bash(jj:*), Bash(git:*), Bash(command:*), Bash(pwd:*), Bash(ls:*), Bash(test:*), Bash(cat:*), Bash(grep:*), Bash(sed:*), Bash(printf:*), Bash(sleep:*), Bash(mkdir:*)
---

# Forgejo with `fj`, `fj-ex`, and Jujutsu

Use this skill whenever the task concerns a Forgejo-hosted repository, issue,
pull request, review, release, workflow, Actions run, job log, artifact, runner,
secret, or variable.

Use the tools in distinct layers:

1. Use **`jj`** for local version-control work when `jj` is installed and the
   current directory belongs to a Jujutsu repository. For a brand-new clone,
   prefer `jj git clone` whenever `jj` is installed unless the user requests Git.
2. Otherwise use **Git** for an existing non-Jujutsu checkout.
3. Use **`fj` as the primary Forgejo client** for Forgejo objects and every
   Actions operation exposed by the installed `fj` build.
4. Use **`fj-ex`** only when the requested Actions capability is absent from
   the installed `fj`.

Do not replace a working `jj` workflow with raw Git merely because `fj` is
designed to work alongside Git. `fj` owns forge operations; `jj` or Git owns
local version control.

## Target CLI

This skill targets the Rust CLI distributed as the `forgejo-cli` crate by
`forgejo-contrib`, whose executable is named `fj`.

Several unrelated tools also use the executable name `fj`. Before relying on
this skill's syntax, verify the installed command surface:

```bash
command -v fj
fj version
fj --help
fj actions --help
```

The expected top-level groups include `repo`, `issue`, `pr`, `wiki`,
`actions`, `release`, `tag`, `user`, `org`, `auth`, and `completion`.

If the installed `fj` has a materially different command tree, treat
`fj <group> --help` as authoritative and adapt rather than guessing. Read
[troubleshooting.md](troubleshooting.md) when the detected CLI or repository
context is ambiguous.

## Reference Map

Load only the reference needed for the task:

| Task | Reference |
| --- | --- |
| Select `jj` or Git; commit, fetch, bookmark, and push safely | [jj-first.md](jj-first.md) |
| Authenticate, select a host/repository, and handle credentials | [authentication.md](authentication.md) |
| Search, inspect, create, comment on, assign, edit, or close issues | [issues.md](issues.md) |
| Inspect, create, review, monitor, or merge pull requests | [pull-requests.md](pull-requests.md) |
| List, dispatch, monitor, diagnose, rerun, or cancel Actions | [actions.md](actions.md) |
| Work with repositories and releases | [repositories-and-releases.md](repositories-and-releases.md) |
| Inspect identity, wikis, organizations, users, and tags | [other-operations.md](other-operations.md) |
| Resolve version, target, authentication, or context failures | [troubleshooting.md](troubleshooting.md) |

## Non-Negotiable Agent Rules

- Detect the local VCS before making local repository changes.
- For an existing checkout, select `jj` only when both `command -v jj` succeeds
  and `jj root` succeeds in the current directory.
- For a brand-new clone into a new destination, prefer `jj git clone` when `jj`
  is installed unless the user explicitly requests a Git-managed clone.
- Do not initialize or convert an existing repository to Jujutsu merely because
  `jj` is installed.
- Once `jj` mode is selected, do not run Git write commands. Use `jj git ...`
  for remotes, fetches, and pushes.
- In `jj` mode, do not use `fj pr checkout`, `fj repo clone`, or repository
  creation options that automatically perform Git writes. Use the Jujutsu
  equivalents.
- Use `fj`, not `curl` or guessed REST endpoints, for normal Forgejo operations.
- Prefer `fj` over `fj-ex` whenever `fj actions --help` shows the required
  operation. Some forks and newer builds add `actions jobs` or `actions logs`
  even when the upstream release with the same version does not.
- Do not install or authenticate `fj-ex` unless the user requests the extended
  operation or it is necessary to complete the requested Actions workflow.
- Read an issue, pull request, release, or run before mutating it.
- Resolve and state the target host and repository before a destructive or
  security-sensitive operation.
- Do not merge, close, delete, cancel, rerun, dispatch, create a release,
  alter a secret, or alter a variable unless it is part of the user's request.
- Preview `fj-ex` cancellation and rerun commands with `--dry-run` before the
  real mutation.
- Never print access tokens, passwords, session cookies, secret values, or
  sensitive log content.
- Treat downloaded Actions logs and artifacts as potentially sensitive.
- Use installed `--help` output when a flag differs from this skill. Never
  invent a flag.

## Start-of-Task Detection

Run this lightweight inspection before repository work:

```bash
pwd
command -v fj
fj version

if command -v jj >/dev/null 2>&1 && jj root >/dev/null 2>&1; then
  printf '%s\n' "VCS_MODE=jj"
  jj --no-pager st
  jj git remote list
elif git rev-parse --show-toplevel >/dev/null 2>&1; then
  printf '%s\n' "VCS_MODE=git"
  git status --short --branch
  git remote -v
else
  printf '%s\n' "VCS_MODE=none"
fi

if command -v fj-ex >/dev/null 2>&1; then
  fj-ex --help >/dev/null
  printf '%s\n' "FJ_EX_AVAILABLE=true"
else
  printf '%s\n' "FJ_EX_AVAILABLE=false"
fi
```

Do not use the shell variable output as persistent global state. It documents
the decision for the current task. Re-run the check after changing directories
or workspaces.

When `fj` cannot infer the Forgejo repository from the current checkout, pass
both the host and repository explicitly:

```bash
fj --host forge.example.com issue search --repo owner/repo --state open
```

For `fj-ex`, use its explicit target options:

```bash
fj-ex actions runs --host forge.example.com --repo owner/repo --limit 20
```

Explicit targeting is especially important for non-colocated Jujutsu
repositories, multiple remotes, forks, and repositories whose primary remote
is not the Forgejo host.

## Responsibility Map

| Operation | Preferred tool |
| --- | --- |
| Clone a new repository | `jj git clone` when `jj` is installed; otherwise `fj repo clone` or Git |
| Local status, diff, history | `jj` in a jj repository; otherwise Git |
| Local commits and history edits | `jj` in a jj repository; otherwise Git |
| Fetch and push | `jj git fetch` / `jj git push` in jj mode; otherwise Git |
| Repository metadata | `fj repo` |
| Issues | `fj issue` |
| Pull requests and reviews | `fj pr` |
| Releases and assets | `fj release` |
| Identity, wikis, organizations, users, and tags | `fj whoami`, `wiki`, `org`, `user`, and `tag` |
| Actions run overview | `fj actions tasks` |
| Workflow dispatch | `fj actions dispatch` |
| Actions variables and secrets | `fj actions variables` / `fj actions secrets` |
| Job list or logs | Built-in `fj actions jobs` / `logs` when present; otherwise `fj-ex` |
| Job watch and artifacts | `fj-ex actions jobs --watch` / `artifacts` unless installed `fj` exposes equivalents |
| Rerun or cancel an Actions run | `fj-ex actions rerun` / `cancel` unless installed `fj` exposes equivalents |

## Default Change-to-PR Workflow

### 1. Establish context

```bash
fj version
fj auth list
```

Then inspect the local repository using the selected VCS. In `jj` mode:

```bash
jj --no-pager st
jj --no-pager log -n 12
jj git remote list
```

In Git mode:

```bash
git status --short --branch
git log --oneline -n 12
git remote -v
```

### 2. Inspect the issue or pull request

Examples:

```bash
fj issue view owner/repo#123
fj pr view owner/repo#45
fj pr view owner/repo#45 files
fj pr view owner/repo#45 diff
```

Read comments, status, and changed files before editing code.

### 3. Make and validate the local change

Use the repository's existing test, lint, formatting, and build commands. Do
not infer that a successful local build means Forgejo Actions will pass.

In `jj` mode, finish a self-contained change with:

```bash
jj --no-pager diff --summary
jj --no-pager diff
# run relevant validation
jj commit -m "Describe the completed change"
jj --no-pager show @- --summary
```

See [jj-first.md](jj-first.md) for bookmarks and pushes.

### 4. Share the change

In `jj` mode:

```bash
jj bookmark create feature-name -r @-
# or move an existing bookmark:
jj bookmark set feature-name -r @-
jj git push --bookmark feature-name
```

Then create or update the Forgejo pull request using `fj`.

In Git mode, use the repository's normal branch and push workflow, then use
`fj pr create`.

### 5. Monitor checks

Start with stock `fj`:

```bash
fj actions --repo owner/repo tasks
fj pr status owner/repo#45 --wait
```

When job-level detail is needed, inspect the installed `fj` first:

```bash
fj actions --help
```

If built-in `jobs` and `logs` are present, prefer them:

```bash
fj actions --repo owner/repo jobs <run-id>
fj actions --repo owner/repo logs --job <job-id>
```

Otherwise, when `fj-ex` is available:

```bash
fj-ex actions runs --repo owner/repo --latest
fj-ex actions jobs --repo owner/repo --latest --watch
```

If a job fails, retrieve only the relevant log, diagnose the failure, fix it,
validate locally, commit and push with the selected VCS, then monitor the new
run. See [actions.md](actions.md).

### 6. Report clearly

At completion, report:

- the host and repository acted on;
- whether local work used `jj` or Git;
- the issue or pull request number;
- the pushed bookmark or branch;
- the Actions result;
- any mutation performed, such as merge, rerun, cancellation, or release;
- any remaining failure or ambiguity.

Do not claim success merely because a command exited without an error. Confirm
the resulting Forgejo object or Actions status.

## Safety Boundaries

Treat these as high-impact operations:

- merging or closing pull requests;
- deleting repositories, releases, tags, branches, or assets;
- cancelling or rerunning workflows;
- dispatching deployment workflows;
- creating or replacing secrets;
- changing repository visibility or permissions;
- force-moving shared bookmarks or force-pushing history.

For a high-impact operation, inspect the target immediately beforehand and
make the target visible in the command. Prefer a dry run when supported.

Do not use `fj-ex auth login` casually. It stores UI credentials and cookies
locally for operations that use Forgejo web endpoints. Read
[authentication.md](authentication.md) before enabling it.

## Version Drift

This skill is written against:

- upstream `forgejo-contrib/forgejo-cli` (`fj`) 0.6.0;
- `forgejo-cli-ex` (`fj-ex`) 0.1.12.

Some downstream `fj` builds add Actions commands without changing the reported
base version. Feature detection is therefore more reliable than version
comparison alone.

Both tools may evolve. Before an unfamiliar or destructive command:

```bash
fj <group> --help
fj <group> <subcommand> --help

fj-ex <group> --help
fj-ex <group> <subcommand> --help
```

Installed help wins over examples in this skill.
