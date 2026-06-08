# Revsets, Filesets, and Templates

jj has three domain-specific languages for querying and formatting: **revsets**
select commits, **filesets** select files, and **templates** format output.

---

## Revsets

A revset is an expression that selects a set of commits. Most `jj` commands
accept revsets via `-r`.

**Always single-quote revset expressions in shell:**

```bash
jj --no-pager log -r 'mine() & ::@'
```

### Symbols

| Symbol            | Meaning                                     |
|-------------------|---------------------------------------------|
| `@`               | Working-copy commit                         |
| `@-`              | Parent of working copy                      |
| `@--`             | Grandparent (repeat `-` for depth)          |
| `<name>@<remote>` | Remote-tracking bookmark (e.g. `main@origin`) |

**Prefer Change IDs** (letters k–z) over Commit IDs (hex). Change IDs survive rewrites.

### Operators (strongest to weakest binding)

| Operator  | Meaning                                           | Example              |
|-----------|---------------------------------------------------|----------------------|
| `x-`      | Parents of x                                      | `@-`                 |
| `x+`      | Children of x                                     | `trunk()+`           |
| `x::`     | Descendants of x (inclusive)                       | `@::`                |
| `::x`     | Ancestors of x (inclusive)                         | `::@`                |
| `x::y`    | Ancestry path: descendants of x that are ancestors of y | `trunk()::@`   |
| `x..`     | Not ancestors of x (complement)                   | `trunk()..`          |
| `..x`     | Ancestors of x excluding root                     | `..@`                |
| `x..y`    | Ancestors of y minus ancestors of x (Git's `x..y`)| `trunk()..@`         |
| `~x`      | Not in x                                          | `~immutable()`       |
| `x & y`   | Intersection                                      | `mine() & ::@`       |
| `x ~ y`   | Difference (in x but not y)                       | `::@ ~ ::trunk()`   |
| `x \| y`  | Union                                             | `bookmarks() \| tags()` |

**`::` vs `..` — the key distinction:**
- `trunk()::@` — ancestry **path**: only commits that are both descendants of trunk AND ancestors of @
- `trunk()..@` — **range**: all ancestors of @ that aren't ancestors of trunk (includes side branches)

### Common Functions

| Function                  | What it selects                                    |
|---------------------------|----------------------------------------------------|
| `mine()`                  | Commits where author email matches current user    |
| `bookmarks()`             | All local bookmark targets                         |
| `trunk()`                 | Head of the default branch (main/master on origin) |
| `description(pattern)`    | Commits with matching description                  |
| `author(pattern)`         | Commits with matching author                       |
| `empty()`                 | Commits modifying no files                         |
| `merges()`                | Merge commits                                      |
| `conflicts()`             | Commits with conflicted files                      |
| `divergent()`             | Divergent changes                                  |
| `files(expression)`       | Commits modifying matching paths                   |
| `diff_lines(text)`        | Commits containing matching diff lines             |
| `heads(x)`                | Commits in x with no descendants in x              |
| `roots(x)`                | Commits in x with no ancestors in x                |
| `ancestors(x, depth)`     | Ancestors with depth limit                         |
| `present(x)`              | Same as x, but returns none() if missing           |

### Practical Recipes

```bash
# My recent work
jj --no-pager log -r 'mine() & trunk()..@'

# Unpushed commits
jj --no-pager log -r 'remote_bookmarks()..@'

# Find commits with "fix" in description
jj --no-pager log -r 'description(substring-i:"fix")'

# Find commits that touched a file
jj --no-pager log -r 'files("src/main.rs")'

# Conflicted commits in my branch
jj --no-pager log -r 'conflicts() & trunk()..@'

# Show the stack I'm working on
jj --no-pager log -r 'reachable(@, mutable())'
```

### String Patterns

| Prefix       | Behavior          | Example                                |
|--------------|-------------------|----------------------------------------|
| `glob:`      | Unix wildcards    | `description(glob:"fix*")`             |
| `substring:` | Contains string   | `description(substring:"TODO")`        |
| `exact:`     | Exact match       | `bookmarks(exact:"main")`              |
| `regex:`     | Regular expression| `author(regex:"^alice")`               |

Append `-i` for case-insensitive: `substring-i:"todo"`.

**Gotcha:** Glob `[` brackets are character classes. Use `substring:` for text
containing brackets.

---

## Filesets

Fileset expressions select files. Many commands accept them as positional arguments.

| Pattern          | Behavior                              | Example                       |
|------------------|---------------------------------------|-------------------------------|
| `prefix-glob:`   | Glob + recursive directory (default) | `jj diff src`                 |
| `glob:`          | Unix glob, cwd-relative              | `jj diff 'glob:"*.rs"'`      |
| `root-glob:`     | Workspace-relative glob              | `jj diff 'root-glob:"**/*.py"'` |

**Operators:** `~x` (not), `x & y` (both), `x | y` (either).

```bash
# All files except Cargo.lock
jj --no-pager diff '~Cargo.lock'

# Rust files in src/
jj --no-pager diff 'src & glob:"**/*.rs"'
```

---

## Templates

Templates customize command output via `-T`.

```bash
# One-line commit summary
jj --no-pager log -r '::@' -T 'change_id.short() ++ " " ++ description.first_line() ++ "\n"'

# Machine-readable IDs
jj --no-pager log --no-graph -T 'commit_id ++ " " ++ change_id ++ "\n"'

# JSON output for scripting
jj --no-pager log --no-graph -r '@' -T 'json(self) ++ "\n"'
```

---

## Useful Config Aliases

```toml
[revset-aliases]
'recent' = 'mine() & trunk()..@'
'conflicted' = 'conflicts() & mutable()'
'unpushed' = 'remote_bookmarks()..@'
'wip' = 'mine() & mutable() & ~empty()'
```

---

## Common Mistakes

- **Unquoted revsets** — `jj log -r mine() & ::@` breaks in shell. Always single-quote.
- **Glob brackets in `description()`** — `[fix]` is a character class. Use `substring:`.
- **Confusing `::` and `..`** — `::` is ancestry path, `..` is range.
- **Forgetting `present()`** — Wrap bookmark references in `present()` if they might not exist.
