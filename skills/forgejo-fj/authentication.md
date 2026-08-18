# Authentication, Hosts, and Repository Context

Use this reference before authenticating, changing credentials, selecting a
Forgejo host, or operating across multiple repositories.

## Verify the Intended `fj`

Several unrelated CLIs are named `fj`. This skill expects the
`forgejo-contrib/forgejo-cli` command tree.

```bash
command -v fj
fj version
fj --help
fj auth --help
fj actions --help
```

The expected command groups include `repo`, `issue`, `pr`, `wiki`, `actions`,
`release`, `tag`, `user`, `org`, `auth`, and `completion`.

If the command tree differs, do not continue by trial and error. Read
[troubleshooting.md](troubleshooting.md).

## Authenticate `fj`

Prefer interactive login so tokens are not copied into a prompt, shell
history, or process arguments:

```bash
fj --host forge.example.com auth login
```

Inspect configured authentication:

```bash
fj auth list
```

Log out only when explicitly requested:

```bash
fj auth logout forge.example.com
```

Before any write, confirm that the authenticated identity has the minimum
required permissions for the target repository.

Do not reveal the token store, token values, OAuth codes, or other credentials
in agent output.

## Select the Host Explicitly

Use `--host` whenever:

- more than one Forgejo instance is configured;
- a repository has multiple forge remotes;
- the current directory does not provide enough repository context;
- a fork and upstream live on different hosts;
- a destructive or security-sensitive operation is about to run.

Example:

```bash
fj --host forge.example.com issue search \
  --repo owner/repo \
  --state open
```

Do not assume `origin` identifies the correct Forgejo instance.

## Select the Repository Explicitly

Prefer an explicit `owner/repo` when the command supports `--repo`:

```bash
fj --host forge.example.com actions \
  --repo owner/repo \
  tasks
```

Issue and pull-request references can also identify the repository:

```bash
fj --host forge.example.com issue view owner/repo#123
fj --host forge.example.com pr view owner/repo#45
```

Explicit references reduce the risk of mutating the wrong repository.

## Jujutsu and Context Inference

Inspect remotes with Jujutsu when Jujutsu mode is active:

```bash
jj git remote list
```

Do not run `git remote` merely to satisfy `fj` if the repository is in
Jujutsu mode.

A non-colocated Jujutsu repository may not expose the Git metadata that `fj`
or `fj-ex` uses for inference. Pass `--host` and `--repo` rather than
initializing or modifying a Git repository.

## `fj-ex` Authentication

`fj-ex` is optional. Check whether it is already available:

```bash
command -v fj-ex
fj-ex --help
```

Use the official `fj` token-backed operations when possible. Authenticate
`fj-ex` only when the requested feature requires its Forgejo web-session
support:

```bash
fj-ex auth login --host forge.example.com
fj-ex auth status --host forge.example.com
```

For non-interactive authentication, consult `fj-ex auth login --help` and use
stdin-based password and one-time-password options rather than command-line
password arguments.

Do not automate `fj-ex auth login` merely to gain convenience. Its stored UI
credentials and cookies permit automatic re-login and are more sensitive than
a narrowly scoped API token.

Log out or clear cookies only when requested or when cleaning up credentials
created specifically for the task:

```bash
fj-ex auth logout --host forge.example.com
fj-ex auth clear-cookies --host forge.example.com
```

## Credential Safety

- Use a dedicated, least-privilege Forgejo account or token for unattended
  agents.
- Do not paste tokens, passwords, one-time passwords, or cookies into issue
  bodies, pull-request comments, logs, commits, or chat output.
- Do not print credential files.
- Do not add credential files to the repository.
- Treat shell tracing (`set -x`) as unsafe around authentication and secrets.
- Treat downloaded Actions logs and artifacts as potentially containing
  secrets.
- Redact sensitive values when reporting a failure.

## Actions Secrets

Stock `fj` 0.6.0 accepts a secret value as command data for
`fj actions secrets create`. That can expose the value through shell history
or process inspection.

Therefore:

- list secret names only when needed;
- never retrieve or print secret values;
- create or replace a secret only when explicitly requested;
- consult installed `fj actions secrets create --help` for a safer stdin or
  file mechanism before passing a value;
- when no safe non-interactive mechanism is available, ask the user to perform
  the secret-value entry manually rather than exposing it.

Variables are not secret. Do not use an Actions variable for confidential
data.

## Authentication Failure Workflow

When a command returns an authentication or authorization error:

1. confirm the host;
2. confirm the repository;
3. run `fj auth list`;
4. identify whether the operation needs read, write, admin, Actions, package,
   or organization permission;
5. do not broaden permissions beyond the requested operation;
6. retry the read operation before retrying a mutation.

Do not repeatedly retry invalid credentials or trigger repeated interactive
login flows.
