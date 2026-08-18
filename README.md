# Jujutsu and Forgejo Agent Skills

This repository contains portable agent skills for Jujutsu and Forgejo
workflows.

- `jj-jujutsu` teaches agents to use Jujutsu (`jj`) instead of raw Git for
  local version-control operations.
- `forgejo-fj` teaches agents to use the Forgejo `fj` CLI for repositories,
  issues, pull requests, releases, and Actions. It is Jujutsu-first when `jj`
  is installed and the current repository is a Jujutsu repository, and it
  prefers `jj git clone` for brand-new clones. It feature-detects any job/log
  support built into `fj`, then uses `fj-ex` as an
  optional extension for missing Actions features such as watching, artifacts,
  reruns, and cancellation.

## Repository Layout

```text
.
├── README.md
└── skills/
    ├── jj-jujutsu/
    │   ├── SKILL.md
    │   ├── config.md
    │   ├── git-interop.md
    │   ├── history.md
    │   ├── pager-commands.md
    │   ├── revsets.md
    │   └── sharing.md
    └── forgejo-fj/
        ├── SKILL.md
        ├── actions.md
        ├── authentication.md
        ├── issues.md
        ├── jj-first.md
        ├── other-operations.md
        ├── pull-requests.md
        ├── repositories-and-releases.md
        └── troubleshooting.md
```

Each installable skill lives in its own directory under `skills/`. The
`name` in each `SKILL.md` frontmatter matches its directory name.

## Install With GitHub CLI

Install the Jujutsu skill globally for Codex:

```bash
gh skill install stevejuma/jj-jujutsu-skill jj-jujutsu \
  --agent codex \
  --scope user
```

Install the Forgejo skill globally for Codex:

```bash
gh skill install stevejuma/jj-jujutsu-skill forgejo-fj \
  --agent codex \
  --scope user
```

For faster installation, install either skill by path:

```bash
gh skill install stevejuma/jj-jujutsu-skill skills/jj-jujutsu \
  --agent codex \
  --scope user
gh skill install stevejuma/jj-jujutsu-skill skills/forgejo-fj \
  --agent codex \
  --scope user
```

To install into a project instead of user scope, replace `--scope user` with
`--scope project`.

## Install With `npx skills`

List the skills discoverable in the repository:

```bash
npx skills add stevejuma/jj-jujutsu-skill --list
```

Install the Jujutsu skill globally for Codex:

```bash
npx skills add stevejuma/jj-jujutsu-skill \
  --skill jj-jujutsu \
  -a codex \
  -g \
  -y
```

Install the Forgejo skill globally for Codex:

```bash
npx skills add stevejuma/jj-jujutsu-skill \
  --skill forgejo-fj \
  -a codex \
  -g \
  -y
```

Install directly from a skill path:

```bash
npx skills add \
  https://github.com/stevejuma/jj-jujutsu-skill/tree/main/skills/forgejo-fj \
  -a codex \
  -g \
  -y
```

## Local Development Install

From the repository root:

```bash
gh skill install . jj-jujutsu --from-local --agent codex --scope user
gh skill install . forgejo-fj --from-local --agent codex --scope user
```

Or with `npx skills`:

```bash
npx skills add . --skill jj-jujutsu -a codex -g
npx skills add . --skill forgejo-fj -a codex -g
```

Restart or reload the agent if it does not immediately discover a newly
installed skill.

## Validation

Verify that both skills are exposed:

```bash
find skills -name SKILL.md -print | sort
```

Expected paths:

```text
skills/forgejo-fj/SKILL.md
skills/jj-jujutsu/SKILL.md
```

Optional checks when the relevant CLIs are available:

```bash
gh skill publish --dry-run
npx skills add . --list
fj version
fj actions --help
command -v jj >/dev/null && jj version
command -v fj-ex >/dev/null && fj-ex --help
```
