# Repositories, Tags, Releases, and Assets

Use `fj` for Forgejo repository metadata and release objects. Keep local
version-control operations in Jujutsu when Jujutsu mode is active.

## Inspect a Repository

Start read-only:

```bash
fj --host forge.example.com repo view owner/repo
fj --host forge.example.com repo readme owner/repo
```

Consult installed help for labels, units, watch, star, and edit operations:

```bash
fj repo --help
fj repo view --help
```

Confirm repository visibility, default branch, clone URLs, and permissions
before creating remotes or pull requests.

## Clone

For a brand-new clone, prefer Jujutsu whenever `jj` is installed unless the
user requests a Git-managed checkout. Do not use `fj repo clone` in that case.
Obtain the clone URL from the repository and use:

```bash
jj git clone <forgejo-repository-url> [destination]
```

Use `fj repo clone` or `git clone` only when the task has selected Git mode.

Do not clone into a non-empty directory without explicit intent.

## Create a Repository

Inspect syntax first:

```bash
fj repo create --help
```

Create the Forgejo repository with `fj`, but avoid automatic Git push options
in Jujutsu mode:

```bash
fj --host forge.example.com repo create new-repo \
  --description "Repository description" \
  --private
```

After remote creation in Jujutsu mode:

```bash
jj git remote add origin <forgejo-repository-url>
jj git push --bookmark main
```

Use the actual trunk bookmark. Do not assume `main` when the repository uses
another default.

Do not add `--push` to `fj repo create` in Jujutsu mode because it crosses
into Git-managed local writes.

## Fork

```bash
fj repo fork owner/repo
```

After forking, inspect the resulting repository and configure remotes
deliberately. In Jujutsu mode:

```bash
jj git remote list
jj git remote add fork <fork-repository-url>
jj git fetch --remote fork
```

Do not replace an existing remote without inspecting it.

## Watch, Star, and Edit

These are Forgejo mutations even though they do not change source history.
Read current state before changing it.

Typical groups include:

```bash
fj repo star --help
fj repo watch --help
fj repo edit --help
```

Repository edits can affect visibility, default branch, features, and
permissions. Use the smallest supported change and verify afterward.

## Delete a Repository

Repository deletion is destructive and normally irreversible through the CLI.
Do not run it unless the user explicitly requests deletion and the exact host,
owner, and repository have been independently verified immediately before the
command.

Use `fj repo delete --help` and follow any confirmation mechanism. Never infer
deletion from phrases such as "clean up" or "remove locally".

## Tags and Jujutsu

Use Jujutsu for local Git-backed history and remote sharing. Use `fj tag` only
for Forgejo-specific tag operations exposed by the installed CLI.

Before creating a release from a tag:

- verify the target commit;
- verify whether the tag already exists;
- verify whether it is local, remote, or both;
- follow the repository's signing policy;
- avoid moving an existing published tag.

Consult both:

```bash
fj tag --help
jj --no-pager log -n 20
```

Do not use raw Git tag writes in Jujutsu mode.

## List and Inspect Releases

```bash
fj release list
fj release view v1.2.3
```

When repository inference is ambiguous, use the target options shown by:

```bash
fj release --help
```

Inspect an existing release before editing or deleting it.

## Create a Release

Check installed syntax:

```bash
fj release create --help
```

Stock `fj` supports release name/tag selection, tag creation, release body,
branch, draft, prerelease, and attachments.

A typical simple release is:

```bash
fj release create v1.2.3 \
  --create-tag \
  --body "Release notes"
```

Before creating:

1. confirm the host and repository;
2. confirm the target commit or branch;
3. confirm the version does not already exist;
4. run release validation;
5. review generated notes;
6. decide deliberately whether this is a draft or prerelease;
7. verify attachments and checksums.

Do not publish a release merely because a build passed. Publication must be
part of the requested task.

## Release Assets

Inspect asset syntax:

```bash
fj release asset --help
fj release asset create --help
fj release asset download --help
fj release asset delete --help
```

Before upload:

- verify the file exists;
- verify it was produced from the intended commit;
- verify its checksum;
- scan for credentials or private configuration;
- avoid uploading local debug artifacts.

Before download, use a temporary or ignored location. Before deletion,
confirm both the release and asset name.

## Edit or Delete a Release

Use installed help:

```bash
fj release edit --help
fj release delete --help
```

Editing release notes is lower risk than changing the associated tag or
artifacts, but it is still a public mutation.

Delete only when explicitly requested. Prefer a draft correction or a new
release when repository policy requires immutable published releases.

## Release Verification

After creation or edit:

```bash
fj release view v1.2.3
```

Verify:

- release name and tag;
- target commit;
- draft/prerelease state;
- body;
- attached assets;
- published URL;
- Actions or deployment result when relevant.

Report the exact release and any assets created. Do not claim an upload
succeeded without viewing the resulting release.
