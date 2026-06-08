# Jujutsu (jj) Configuration

Configuration reference for jj — config file locations, agent-specific setup,
useful aliases, and customization.

---

## Config Precedence (later overrides earlier)

1. **Built-in** — compiled into jj
2. **User** — `jj config edit --user`
3. **Repo** — `jj config edit --repo`
4. **Workspace** — `jj config edit --workspace`
5. **Command-line** — `--config name=value` or `--config-file path`

```bash
# Find your config file
jj --no-pager config path --user

# List all active config
jj --no-pager config list

# Set a value
jj config set --user ui.pager "less -FRX"
```

The `JJ_CONFIG` environment variable overrides all default user config locations.

---

## Agent-Specific Configuration

Agents should use dedicated config to prevent editor hangs and pager blocking:

```toml
# agent-jj-config.toml
[user]
name = "Agent"
email = "agent@example.com"

[ui]
editor = "TRIED_TO_RUN_AN_INTERACTIVE_EDITOR"
diff-formatter = ":git"
paginate = "never"
```

Launch with: `JJ_CONFIG=/path/to/agent-jj-config.toml <agent-harness>`

### Key Settings for Agents

| Setting             | Value                                    | Why                                    |
|---------------------|------------------------------------------|----------------------------------------|
| `ui.paginate`       | `"never"`                                | Prevents pager from blocking           |
| `ui.editor`         | Non-interactive string                   | Errors clearly if editor is invoked    |
| `ui.diff-formatter` | `":git"`                                 | Machine-parseable diffs                |

### Quick One-Time Repo Setup

If you can't use a config file, set the critical values per-repo:

```bash
jj config set --repo ui.paginate never
```

---

## Useful Aliases

```toml
[aliases]
l = ["log", "-r", "(main..@):: | (main..@)-"]
mine = ["log", "-r", "mine()"]
conflicts = ["log", "-r", "conflicted()"]
empties = ["log", "-r", "empty() & mine()"]
d = ["diff"]
```

---

## Revset Aliases

```toml
[revset-aliases]
"immutable_heads()" = "builtin_immutable_heads() | release@origin"
```

Default `jj log` revset:

```toml
[revsets]
log = "main@origin.."
```

---

## Diff Format

```toml
[ui]
# Built-in options: ":color-words" (default), ":git", ":summary", ":stat", ":name-only"
diff-formatter = ":git"
```

---

## Commit Signing

```toml
[signing]
behavior = "own"       # "drop" | "keep" | "own" | "force"
backend = "ssh"        # "gpg" | "ssh" | "none"
key = "ssh-ed25519 AAAAC3..."
```

---

## Auto-Track for Remotes

```toml
[remotes.origin]
auto-track-bookmarks = "*"              # Track all bookmarks

[remotes.upstream]
auto-track-bookmarks = "main"           # Only track main from upstream
```

---

## Conditional Config

Apply different settings per repository or command:

```toml
# Override email for OSS repos
[[--scope]]
--when.repositories = ["~/oss"]
[--scope.user]
email = "oss@example.org"

# Use delta only for diff/show commands
[[--scope]]
--when.commands = ["diff", "show"]
[--scope.ui]
pager = "delta"
```

---

## Common Config Mistakes

| Symptom                    | Cause                      | Fix                                        |
|----------------------------|----------------------------|--------------------------------------------|
| Config not taking effect   | Wrong precedence level     | Check with `jj --no-pager config list`     |
| Editor opens unexpectedly  | `ui.editor` not overridden | Set to a non-interactive string            |
| Pager blocks execution     | Pager waiting for input    | Set `ui.paginate = "never"`                |
| Bookmarks not tracking     | Too-restrictive pattern    | Check `remotes.<name>.auto-track-bookmarks`|
