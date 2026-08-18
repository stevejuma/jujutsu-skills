# Identity, Wikis, Organizations, and Users

Use this reference for stock `fj` operations outside the core repository,
issue, pull-request, release, and Actions workflows.

Prefer read-only commands. Organization, account, key, and visibility changes
can affect access across many repositories and require explicit authorization.

## Confirm the Current Identity

```bash
fj --host forge.example.com whoami
```

Run this before an account-sensitive operation when more than one host or
identity may be configured.

Do not infer the acting identity only from a local Git or Jujutsu author
configuration. Forgejo API authentication is separate.

## Wikis

List wiki pages:

```bash
fj wiki --repo owner/repo contents
```

View a page:

```bash
fj wiki --repo owner/repo view Home
```

Open a page in the browser:

```bash
fj wiki --repo owner/repo browse Home
```

Use an explicit host when needed:

```bash
fj --host forge.example.com wiki --repo owner/repo view Home
```

Stock `fj` can clone a wiki repository, but `fj wiki clone` performs a Git
clone internally. In a Jujutsu-first workflow:

- prefer API-based `contents` and `view` for reading;
- when a local wiki checkout is required, confirm the `.wiki.git` URL and use
  `jj git clone`;
- use `fj wiki clone` only when the task selected Git mode or the user
  explicitly accepts a separate Git-managed wiki checkout.

Do not assume that editing a local wiki checkout updates the main source
repository. A Forgejo wiki is a separate repository.

## Organizations: Read-Only Operations

List organizations:

```bash
fj --host forge.example.com org list
fj --host forge.example.com org list --only-member-of
```

Inspect an organization and membership:

```bash
fj --host forge.example.com org view example-org
fj --host forge.example.com org members example-org
fj --host forge.example.com org repo list example-org
```

Inspect activity only when it is relevant:

```bash
fj --host forge.example.com org activity example-org
```

Organization output can reveal private membership or repository information.
Do not copy it into public issues or pull requests.

## Create an Organization Repository

For an organization-owned repository:

```bash
fj --host forge.example.com org repo create example-org new-repo \
  --description "Repository description" \
  --private
```

In Jujutsu mode, do not use `--remote` or `--push`; those options manipulate
the local repository through Git. Create the remote repository first, then add
and push the remote with `jj git remote add` and `jj git push`.

## Organization Mutations

Stock `fj` also supports organization creation, editing, member visibility,
teams, labels, and repositories.

Before any such mutation:

```bash
fj org --help
fj org team --help
fj org label --help
fj org repo --help
```

Treat these as high-impact:

- creating or editing an organization;
- changing organization visibility;
- changing a member's public/private visibility;
- adding or removing team members;
- changing team permissions or repository access;
- creating, replacing, or deleting organization labels;
- creating repositories under an organization.

Read the current organization, team, member, label, or repository state
immediately before changing it. Verify afterward.

## Users: Search and Inspect

```bash
fj --host forge.example.com user search "display name or username"
fj --host forge.example.com user view username
fj --host forge.example.com user repos username
fj --host forge.example.com user orgs username
fj --host forge.example.com user activity username
```

A user argument omitted from some commands means the authenticated user.
Specify it when ambiguity could change the result.

## User and Account Mutations

Stock `fj` supports follow, unfollow, block, unblock, profile edits, SSH keys,
and GPG keys.

Before an account mutation:

```bash
fj user --help
fj user edit --help
fj user key --help
fj user gpg --help
```

Rules:

- do not follow, unfollow, block, or unblock a user without an explicit request;
- do not edit profile fields as a side effect of repository work;
- inspect key IDs and fingerprints before deletion;
- never print private key material;
- upload only public SSH or GPG key material;
- do not use a force or no-verification option unless the user requested and
  understood it;
- verify the authenticated account with `fj whoami` first.

Key deletion can remove repository access or signature verification. Treat it
as destructive.

## Tags

List and inspect Forgejo tags:

```bash
fj tag --repo owner/repo list
fj tag --repo owner/repo view v1.2.3
```

Create only when requested and after verifying the target:

```bash
fj tag --repo owner/repo create v1.2.3 --branch main
```

Use the actual target branch or commit policy. Do not assume `main`.

Delete only with explicit authorization:

```bash
fj tag --repo owner/repo delete v1.2.3
```

A published tag may be referenced by releases, package metadata,
deployments, or external consumers. Inspect those relationships before
deleting or recreating it.

## Completion and Help

Generate shell completion only when requested:

```bash
fj completion --help
```

For any operation not covered here, use the installed command tree:

```bash
fj --help
fj <group> --help
fj <group> <subcommand> --help
```

Do not fall back to guessed REST endpoints merely because a less-common
`fj` subcommand is unfamiliar.
