# History Rewriting and Investigation

Covers squashing, absorbing, rebasing, splitting commits (agent-safe), conflict
resolution, investigating history, and cleanup.

---

## Curating Commits

### Squash: Fold Changes Into Parent

```bash
# Squash all changes from @ into @-
jj squash -m "feat: complete user validation"

# Squash specific files only
jj squash src/auth.rs src/auth_test.rs

# Squash from a specific source into a specific destination
jj squash --from <change-id> --into <target-change-id>
```

**Agent rule:** Always use `-m` when squashing — omitting it opens an editor.

After squashing, `@` becomes empty (all changes moved to parent). Either start
new work with `jj new` or abandon the empty commit.

### Absorb: Auto-Distribute Changes

`jj absorb` automatically distributes each hunk in the working copy to the
ancestor commit that last modified those lines.

```bash
# Absorb all working-copy changes into appropriate ancestors
jj absorb

# Verify the result
jj --no-pager log -r 'ancestors(@, 5)'

# Undo if it wasn't right
jj undo
```

**When to use absorb vs squash:**
- **absorb** — working copy has changes to lines touched by different ancestor commits
- **squash** — all changes belong in one parent commit

### Rebase: Move Commits

```bash
# Rebase current commit and descendants onto trunk
jj rebase -s @ -o trunk()

# Rebase a single commit (not its descendants)
jj rebase -r <change-id> -o trunk()

# Rebase a bookmark's branch onto trunk
jj rebase -b my-feature -o trunk()
```

**Flag meanings:**

| Flag         | What it rebases                                              |
|--------------|--------------------------------------------------------------|
| `-s <rev>`   | The revision AND all its descendants                         |
| `-r <rev>`   | Only that single revision (descendants rebase onto its parent) |
| `-b <rev>`   | The entire branch containing rev                             |
| `-o <dest>`  | Destination (what to rebase onto). Use `-o`, not deprecated `-d` |

**After rebase:** always `jj --no-pager st` to check for conflicts.

---

## Splitting Commits

### Agent-Safe Splitting

`jj split` without file paths is interactive and **will hang**. Two safe approaches:

**Approach 1: Split by file paths (non-interactive)**

```bash
# Split specific files out of a commit
# Named files go into the first commit, everything else stays in the second
jj split -r <change-id> src/auth.rs src/auth_test.rs -m "feat: add auth module"
```

**Approach 2: The restore workflow (any boundary)**

When you need to split within a file or the boundary is complex:

```bash
# 1. Create a new commit on top of the one to split
jj new <change-id> -m "part 2: error handling"

# 2. Restore (copy) only the files you DON'T want in part 2
jj restore --from <change-id>- src/validation.rs

# 3. Describe the original commit
jj describe -r <change-id> -m "part 1: input validation"

# 4. Verify both commits
jj --no-pager show <change-id>
jj --no-pager show @
```

---

## Conflict Resolution

jj records conflicts in commits instead of blocking operations. A rebase or merge
that produces conflicts still succeeds — the conflicted state is stored and you
resolve when ready.

### Identifying Conflicts

```bash
jj --no-pager st                                    # check for conflicts
jj --no-pager log -r 'conflicts() & trunk()..@'     # find all conflicted commits
jj --no-pager show <change-id>                       # see conflicted files
```

### Resolution Workflow

**Method 1: Edit directly (simple conflicts)**

```bash
jj edit <conflicted-change-id>
# Edit the conflicted file — remove ALL conflict markers
jj --no-pager st    # should show no conflicts
```

**Method 2: New commit + squash (safer for complex conflicts)**

```bash
jj new <conflicted-change-id>
# Edit files to resolve conflicts
jj squash -m "resolve conflicts"
```

### Conflict Marker Format

jj uses a diff-based marker format different from Git:

```text
<<<<<<< conflict 1 of 1
%%%%%%% diff from base to side 1
 unchanged line
-removed in side 1
+added in side 1
+++++++ side 2 content
side 2 full content here
>>>>>>> conflict 1 of 1 ends
```

### Agent Conflict Resolution Rules

1. **Never use `jj resolve`** — it opens an interactive merge tool
2. **Edit conflict markers directly** in the file
3. **Remove ALL marker lines** (`<<<<<<<`, `>>>>>>>`, `%%%%%%%`, `+++++++`)
4. **Verify with `jj --no-pager st`** — output should show no conflict warnings
5. **Auto-rebase propagation** — resolving a parent conflict automatically re-resolves
   descendants that inherited it

---

## Investigating History

```bash
# Show a specific commit's changes
jj --no-pager show <change-id>

# View diff of a specific file
jj --no-pager diff -r <change-id> src/main.rs

# View file content at a revision (without switching)
jj --no-pager file show -r <change-id> src/main.rs

# Who last changed each line (like git blame)
jj --no-pager file annotate src/main.rs

# Find commits that touched a specific file
jj --no-pager log -r 'files("src/main.rs")'

# Search commit messages
jj --no-pager log -r 'description(substring-i:"auth")'

# See how a change evolved over time
jj --no-pager evolog -r <change-id>

# Operation history
jj --no-pager op log
```

---

## Abandoning and Cleanup

```bash
# Abandon a specific commit (descendants rebase onto its parent)
jj abandon <change-id>

# Abandon multiple commits
jj abandon <id1> <id2> <id3>

# Find empty commits in your branch
jj --no-pager log -r 'empty() & trunk()..@'

# Abandon all empty commits in your branch
jj abandon 'empty() & mine() & trunk()..@'

# Revert a commit (create reverse changes without removing from history)
jj revert -r <change-id>
```

---

## Verification Checklist

After any major history rewrite (split, rebase, large squash):

1. **No conflicts:** `jj --no-pager log -r 'conflicts() & trunk()..@'` — should return nothing
2. **No unintended empties:** `jj --no-pager log -r 'empty() & trunk()..@'` — review results
3. **Commits are focused:** `jj --no-pager show <id> --stat` for each rewritten commit
4. **Messages are clear:** `jj --no-pager log -r 'trunk()..@'` — check descriptions
5. **Bookmarks correct:** `jj --no-pager bookmark list` — verify positions
6. **Status clean:** `jj --no-pager st` — no unexpected state

---

## Common Mistakes

- **Using `jj split` without file paths** — hangs waiting for interactive input
- **Using `jj squash -i`** — interactive, hangs agents
- **Forgetting `-m` on squash** — opens an editor
- **Not checking for conflicts after rebase** — rebase succeeds even with conflicts
- **Not using `jj undo` for recovery** — if a split, squash, or rebase goes wrong,
  `jj undo` immediately reverses it
