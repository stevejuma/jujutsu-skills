# Forgejo Actions with `fj` and `fj-ex`

Use `fj` first and inspect its installed command surface. Upstream
`forgejo-contrib/forgejo-cli` 0.6.0 exposes task overview, dispatch, variables,
and secrets. Some downstream builds add built-in job and log commands without
changing the base version. Use those built-ins when present, then use `fj-ex`
only for capabilities the installed `fj` lacks.

## Capability Split

| Capability | Preferred command |
| --- | --- |
| Recent Actions task overview | `fj actions tasks` |
| Workflow dispatch | `fj actions dispatch` |
| Variables | `fj actions variables` |
| Secrets | `fj actions secrets` |
| Filtered run list | `fj-ex actions runs`, unless installed `fj` offers an equivalent |
| Job list | Built-in `fj actions jobs` when present; otherwise `fj-ex actions jobs` |
| Job watch | `fj-ex actions jobs --watch`, unless installed `fj` offers an equivalent |
| Job or run logs | Built-in `fj actions logs` when present; otherwise `fj-ex actions logs` |
| Artifact list/download | `fj-ex actions artifacts`, unless installed `fj` offers an equivalent |
| Cancel | `fj-ex actions cancel`, unless installed `fj` offers an equivalent |
| Rerun | `fj-ex actions rerun`, unless installed `fj` offers an equivalent |
| Workflow inventory | `fj-ex actions workflows` |
| Runner tokens and queued jobs | `fj-ex actions runners` |

Do not use `fj-ex actions trigger` when stock
`fj actions dispatch` satisfies the request.

## Stock `fj`: Task Overview

```bash
fj --host forge.example.com actions \
  --repo owner/repo \
  tasks
```

Paginate when necessary:

```bash
fj actions --repo owner/repo tasks --page 2
```

Match the run to the intended workflow, event, branch, and commit. Do not
assume the newest visible run belongs to the change being diagnosed.

## Stock `fj`: Dispatch

```bash
fj actions --repo owner/repo dispatch ci.yml main
```

With workflow inputs:

```bash
fj actions --repo owner/repo dispatch deploy.yml main \
  -I environment=staging \
  -I dry_run=true
```

Dispatch is a mutation and may deploy software or consume runner resources.
Before dispatching, verify the workflow name, ref, inputs, host, and
repository.

Do not dispatch a production workflow unless the user's request clearly
authorizes it.

## Stock `fj`: Variables

```bash
fj actions --repo owner/repo variables list
fj actions --repo owner/repo variables list --verbose
fj actions --repo owner/repo variables create NAME VALUE
fj actions --repo owner/repo variables create NAME VALUE --force
fj actions --repo owner/repo variables delete NAME
```

Variables are not secrets. Do not store confidential data in them.

Create, replace, or delete a variable only when requested. Read the current
list first. Use `--force` only when intentional replacement is required.

## Stock `fj`: Secrets

```bash
fj actions --repo owner/repo secrets list
```

Stock `fj` 0.6.0 also supports `create` and `delete`, but secret creation takes
secret data as command input. Read [authentication.md](authentication.md)
before automating it.

Never print or attempt to read secret values. Listing should expose names, not
values.

## Feature-Detect Built-in Jobs and Logs

Do not decide from the version string alone:

```bash
fj actions --help
```

When the installed build lists `jobs` and `logs`, prefer them over `fj-ex`:

```bash
fj actions --repo owner/repo jobs <run-id>
fj actions --repo owner/repo logs --job <job-id>
fj actions --repo owner/repo logs \
  --run <run-id> \
  --out .tmp/forgejo-logs/run-<run-id>.zip
```

Confirm exact syntax with:

```bash
fj actions jobs --help
fj actions logs --help
```

Built-in job/log support may still depend on the Forgejo server version. If the
command exists but the server rejects the endpoint, fall back to `fj-ex` only
when it supports that server and the user has authorized any required
authentication.

## Extended Monitoring with `fj-ex`

Check availability:

```bash
command -v fj-ex
fj-ex --help
```

List recent runs:

```bash
fj-ex actions runs --repo owner/repo --limit 20
fj-ex actions runs --repo owner/repo --latest
fj-ex actions runs --repo owner/repo --status failure
fj-ex actions runs --repo owner/repo --workflow ci.yml
```

Watch jobs:

```bash
fj-ex actions jobs --repo owner/repo --latest --watch
```

Whenever practical, filter by workflow and identify the exact run before
using `--latest`. A newer unrelated run may otherwise be selected.

## Diagnose a Failure

Start with `fj actions --help`. If the installed `fj` has built-in jobs
and logs, use those after identifying a run from `fj actions tasks`.

Otherwise start with an `fj-ex` run and job overview:

```bash
fj-ex actions runs --repo owner/repo --workflow ci.yml --limit 20
fj-ex actions jobs --repo owner/repo --latest
```

Retrieve only the relevant job log:

```bash
fj-ex actions logs job \
  --repo owner/repo \
  --latest \
  --job-index 0
```

For all logs in a run:

```bash
fj-ex actions logs run \
  --repo owner/repo \
  --run-index 50 \
  --out-dir .tmp/forgejo-logs/run-50
```

Diagnosis workflow:

1. confirm the run's workflow and commit;
2. identify the failed job;
3. retrieve the smallest relevant log;
4. redact tokens, credentials, personal data, and private endpoints;
5. identify the first causal error rather than only the final cascade;
6. correlate it with the local diff and workflow definition;
7. make and validate the fix;
8. commit and push using the VCS selected in [jj-first.md](jj-first.md);
9. monitor the new run;
10. report the exact result.

Do not commit downloaded logs.

## Artifacts

List:

```bash
fj-ex actions artifacts list --repo owner/repo --latest
```

Download a named artifact:

```bash
fj-ex actions artifacts get \
  --repo owner/repo \
  --run-index 50 \
  --artifact my-artifact \
  --out-file .tmp/my-artifact.zip
```

Before downloading, confirm the artifact name, run, and expected sensitivity.
Store it in a temporary or ignored directory. Do not commit it unless the task
explicitly requires publishing that artifact.

## Rerun

Preview first:

```bash
fj-ex actions rerun \
  --repo owner/repo \
  --run-index 50 \
  --dry-run
```

Then, only when authorized:

```bash
fj-ex actions rerun \
  --repo owner/repo \
  --run-index 50
```

Rerun only failed jobs when appropriate:

```bash
fj-ex actions rerun \
  --repo owner/repo \
  --latest \
  --failed-only \
  --dry-run
```

After reviewing the preview, remove `--dry-run`.

Do not rerun repeatedly without diagnosing a deterministic failure. A rerun is
appropriate for a suspected transient runner, network, or service failure,
not as a substitute for fixing code.

## Cancel

Preview:

```bash
fj-ex actions cancel \
  --repo owner/repo \
  --run-index 50 \
  --dry-run
```

Then, only when requested:

```bash
fj-ex actions cancel \
  --repo owner/repo \
  --run-index 50
```

Confirm that the run is still active and belongs to the intended repository
and commit. Cancellation can interrupt deployments and leave partial external
state.

## Workflow and Runner Inspection

List workflows:

```bash
fj-ex actions workflows --repo owner/repo
```

Inspect queued runner jobs:

```bash
fj-ex actions runners jobs --repo owner/repo --waiting
```

Runner registration tokens and global runner operations are sensitive. Use
them only for an explicit runner-administration task and do not print tokens
in the final response.

## Monitoring Without `fj-ex`

When `fj-ex` is unavailable:

1. inspect `fj actions --help` for built-in jobs and logs;
2. use available built-ins;
3. use `fj actions tasks` for an overview;
4. use `fj pr status <pr> --wait` for pull-request checks when applicable;
5. if polling is necessary, use a bounded loop with a reasonable interval;
6. stop after a clear terminal state or bounded number of attempts;
7. report exactly which job, log, artifact, rerun, or cancellation capability
   remained unavailable.

Do not run an unbounded polling loop.

## Credential and Data Warning

Some `fj-ex` features use Forgejo web endpoints and locally stored UI
credentials or cookies. Downloaded logs and artifacts can contain secrets.
Use least-privilege accounts, temporary ignored paths, and careful redaction.
Do not include sensitive log excerpts in issues or pull-request comments.
