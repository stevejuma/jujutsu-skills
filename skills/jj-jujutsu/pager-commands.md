# Pager Commands Reference

jj uses a pager (typically `less`) that **will hang agent/non-interactive environments**.
When `ui.paginate` is set to `"auto"` (the default), jj pipes output through the pager
for commands that produce potentially long output.

**Installed version**: jj 0.39.0
**Default pager**: `less` (inherited from `$PAGER` environment variable)
**Default `ui.paginate`**: `auto`

---

## The Problem

When `ui.paginate = "auto"`, jj activates the pager for any command that produces
output longer than the terminal height. The pager waits for user input (q, space,
arrow keys) to continue — an agent cannot provide this input, causing the process
to hang indefinitely.

The `--no-pager` flag is a **global option** available on every jj command.

---

## Solution: Disable the Pager

### Option 1: `--no-pager` flag (recommended for agents)

Add `--no-pager` as a global flag to every `jj` invocation:

```bash
jj --no-pager log
jj --no-pager diff
jj --no-pager show @
jj --no-pager status
```

**Note**: `--no-pager` must come BEFORE the subcommand name.

### Option 2: Repository config (one-time setup)

```bash
jj config set --repo ui.paginate never
```

### Option 3: User config (affects all repos)

```bash
jj config set --user ui.paginate never
```

### Option 4: Environment variable

```bash
JJ_PAGER=cat jj log
# or
export JJ_PAGER=cat
```

### Option 5: Agent config file

Create a dedicated agent config:

```toml
# agent-jj-config.toml
[ui]
paginate = "never"
editor = "TRIED_TO_RUN_AN_INTERACTIVE_EDITOR"
```

Launch with: `JJ_CONFIG=/path/to/agent-jj-config.toml <agent-command>`

---

## Commands That Produce Paginated Output

The `--no-pager` global flag is accepted by **all** jj commands. However, the following
commands are the ones most likely to trigger the pager in practice because they
produce multi-line output:

### High Risk (commonly produce long output)

| Command            | Why it pagers                                    |
|--------------------|--------------------------------------------------|
| `jj log`           | Revision history — can be very long              |
| `jj diff`          | File diffs — grows with change size              |
| `jj show`          | Commit details + diff                            |
| `jj evolog`        | Evolution log for a change                       |
| `jj interdiff`     | Diff between two revisions' diffs                |
| `jj op log`        | Operation history — grows over time              |
| `jj file show`     | File contents at a revision                      |
| `jj file annotate` | Blame-style line annotations                     |
| `jj config list`   | Lists all config values                          |
| `jj help`          | Help text for commands                           |

### Medium Risk (may page with enough items)

| Command              | Why it pagers                                  |
|----------------------|------------------------------------------------|
| `jj status`          | Normally short, but many changed files = long  |
| `jj bookmark list`   | With many bookmarks                            |
| `jj tag list`        | With many tags                                 |
| `jj workspace list`  | With many workspaces                           |

### Low Risk (short output, but still respect pager)

| Command        | Notes                                              |
|----------------|----------------------------------------------------|
| `jj version`   | Single line — unlikely to page                     |
| `jj root`      | Single line — unlikely to page                     |
| `jj st`        | Usually short                                      |

---

## Safe Command Patterns for Agents

Always prefix with `--no-pager`:

```bash
# Status check
jj --no-pager st

# View log (limit output with -n for extra safety)
jj --no-pager log -n 20

# View diff
jj --no-pager diff

# Show a specific commit
jj --no-pager show <change-id>

# File contents
jj --no-pager file show -r <change-id> path/to/file

# Operation history
jj --no-pager op log -n 10

# Bookmark list
jj --no-pager bookmark list

# Config listing
jj --no-pager config list
```

**Tip**: For `jj log`, consider using `-n <count>` to limit the number of revisions
shown, reducing output even further:

```bash
jj --no-pager log -n 10 -r 'trunk()..@'
```

---

## Commands That Are Safe WITHOUT `--no-pager`

These commands produce no output or only write to the repo (no stdout to page):

| Command          | Why it's safe                          |
|------------------|----------------------------------------|
| `jj new`         | Creates commit, minimal output         |
| `jj describe`    | Sets description, brief confirmation   |
| `jj squash`      | Mutates history, brief output          |
| `jj abandon`     | Removes commit, brief output           |
| `jj undo`        | Reverses operation, brief output       |
| `jj restore`     | Restores files, brief output           |
| `jj rebase`      | Moves commits, brief output            |
| `jj edit`        | Changes working copy, brief output     |
| `jj bookmark set`| Moves bookmark, brief output           |
| `jj git push`    | Pushes to remote, brief output         |
| `jj git fetch`   | Fetches from remote, brief output      |
| `jj commit`      | Finalizes commit, brief output         |

**However**, it's safest to always use `--no-pager` anyway — the overhead is zero
and it prevents surprises if a command's output grows unexpectedly.
