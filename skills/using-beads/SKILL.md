---
name: using-beads
description: Use when tracking work across sessions, managing task dependencies, preventing forgotten work, or finding ready tasks in long-horizon projects - dependency-aware issue tracker for AI agents providing persistent memory and blocking relationships
---

# Using Beads Issue Tracker

## Overview

**Beads is a memory upgrade for coding agents.** It's a dependency-aware issue tracker designed for AI agents to track work across sessions, manage complex task hierarchies, and always know what's ready to work on next.

**Core principle:** Agents lose context between sessions. Beads provides persistent memory so you never forget tasks, can track discovered work, and always find unblocked issues ready to execute.

**Installation:** https://github.com/steveyegge/beads

**Run `bd quickstart` for interactive tutorial. Use `bd --help` or `bd <command> --help` for complete command documentation.**

## When to Use

Use Beads for:
- **Long-horizon projects** - Work spanning multiple sessions
- **Complex dependencies** - Tasks blocking other tasks
- **Discovered work** - Issues found while executing
- **Epic planning** - Parent tasks with subtasks
- **Team coordination** - Multiple agents working on same project
- **Persistent memory** - Never lose track of work

**Symptoms indicating you need Beads:**
- Forgetting tasks between sessions
- Duplicating work already planned
- Starting tasks before dependencies complete
- Losing track of discovered bugs/improvements
- Unclear what to work on next
- Difficulty coordinating parallel work

## Core Concepts

### Four Dependency Types

Beads supports four relationship types between issues:

**1. blocks** - Hard blocker (affects ready work)
```bash
bd dep add bd-5 bd-3 --type blocks  # bd-5 blocked by bd-3
```
Use when issue X cannot start until Y completes.

**2. related** - Soft relationship (context only)
```bash
bd dep add bd-10 bd-8 --type related  # bd-10 related to bd-8
```
Use for connected issues that don't block each other.

**3. parent-child** - Epic/subtask hierarchy
```bash
bd dep add bd-15 bd-12 --type parent-child  # bd-15 is child of epic bd-12
```
Use for epic decomposition into subtasks.

**4. discovered-from** - Track discovery during execution
```bash
bd create "Fix edge case" -t bug -p 1 --deps discovered-from:bd-20
```
Use when discovering new work while executing another task.

### Hierarchical Blocking

**CRITICAL:** Blocking propagates through parent-child hierarchies.

When a parent (epic) is blocked, ALL children are automatically blocked, even if they have no direct blockers.

```bash
# Epic blocked → all children blocked
bd create "Epic: Auth" -t epic
bd create "Add login" -t task
bd dep add bd-2 bd-1 --type parent-child  # bd-2 child of bd-1
bd create "Design auth" -t task -p 0
bd dep add bd-1 bd-3 --type blocks  # bd-1 blocked by bd-3

# Now BOTH bd-1 and bd-2 are blocked
bd ready  # Neither shows up
bd blocked  # Shows both
```

Blocking propagates up to 50 levels deep through parent-child relationships.

### Ready Work Algorithm

An issue is "ready" if:
- Status is `open`
- Has NO open `blocks` dependencies
- All blockers are either closed or non-existent
- Not blocked by parent hierarchy

Only `blocks` and inherited parent blocking affect ready status. `related` and `discovered-from` do NOT affect ready work.

### Database Discovery

bd automatically discovers databases in this order:
1. `--db` flag: `bd --db /path/to/db.db list`
2. `$BEADS_DB` environment variable
3. `.beads/*.db` in current directory or ancestors (walks up like git)
4. `~/.beads/default.db` as fallback

**Multi-project isolation:** Each project gets its own `.beads/*.db` database. bd finds the correct one based on current directory.

### Automatic Sync

bd automatically syncs between SQLite (fast queries) and JSONL (git-friendly):
- **Auto-export:** After CRUD operations (5-second debounce)
- **Auto-import:** When JSONL newer than DB (e.g., after git pull)

Manual sync:
```bash
bd export -o .beads/issues.jsonl  # Force export
bd import -i .beads/issues.jsonl  # Force import
```

## Command Reference

### Initialization

#### bd init

Initialize Beads in a project.

```bash
bd init                           # Creates .beads/beads.db
bd init --prefix myapp            # Creates .beads/myapp.db
bd init --db /path/to/custom.db   # Custom location
```

**Flags:**
- `--prefix PREFIX` - Database filename prefix (default: beads)
- `--db PATH` - Custom database path

Creates:
- `.beads/` directory
- SQLite database (gitignored)
- Initial `.beads/issues.jsonl` (committed to git)

**Best practice:** Run once per project root.

### Creating Issues

#### bd create

Create new issues.

```bash
# Basic creation
bd create "Fix login bug"
bd create "Add feature" -d "Description" -p 1 -t feature

# With all options
bd create "Task title" \
  -d "Long description" \
  -p 2 \
  -t task \
  -a alice \
  -l "backend,urgent" \
  --id custom-123 \
  --json

# Create with dependencies (single command)
bd create "Fix bug" -t bug -p 1 --deps discovered-from:bd-20

# From markdown file
bd create -f feature-plan.md
```

**Flags:**
- `-d, --description TEXT` - Issue description
- `-p, --priority INT` - Priority (0-4, where 0=highest, 4=lowest)
- `-t, --type TYPE` - Type: bug, feature, task, epic, chore
- `-a, --assignee USER` - Assign to user
- `-l, --labels LIST` - Comma-separated labels
- `--id ID` - Explicit issue ID (e.g., worker1-100 for parallel workers)
- `--deps TYPE:ID` - Create dependency in same command (e.g., discovered-from:bd-5)
- `-f, --file PATH` - Create multiple issues from markdown
- `--json` - Output in JSON format

**Markdown file format:**
```markdown
## Issue Title

Optional description text.

### Priority
1

### Type
feature

### Description
Detailed description (overrides text after title).

### Design
Design notes.

### Acceptance Criteria
- Must do this
- Must do that

### Assignee
username

### Labels
label1, label2

### Dependencies
bd-10, bd-20
```

**JSON output:**
```bash
bd create "Fix bug" --json | jq -r '.id'
```

**Best practices:**
- Use descriptive titles (not "Fix issue" or "Update code")
- Set priority appropriately (P0=critical, P1=high, P2=normal)
- Add labels for categorization
- Use `--json` in agent scripts for parsing
- Use explicit IDs when parallel agents might create issues simultaneously

### Viewing Issues

#### bd show

Show full details of an issue.

```bash
bd show bd-1
bd show bd-1 --json
```

**Flags:**
- `--json` - Output in JSON format

**Output includes:**
- All fields (title, description, status, priority, type)
- Dependencies (blockers, blocked by)
- Timestamps (created, updated, closed)
- Audit trail

#### bd list

List issues with filtering.

```bash
# List all
bd list
bd list --json

# Filter by status
bd list --status open
bd list --status in_progress
bd list --status closed
bd list --status blocked

# Filter by priority
bd list --priority 0        # P0 only
bd list --priority 1        # P1 only

# Filter by assignee
bd list --assignee alice

# Filter by type
bd list --type bug
bd list --type feature
bd list --type task
bd list --type epic
bd list --type chore

# Filter by labels
bd list --label urgent
bd list --label backend

# Combine filters
bd list --status open --priority 0 --type bug --json
```

**Flags:**
- `--status STATUS` - Filter by status (open, in_progress, closed, blocked)
- `--priority INT` - Filter by priority (0-4)
- `--assignee USER` - Filter by assignee
- `--type TYPE` - Filter by type
- `--label LABEL` - Filter by label (repeatable)
- `--json` - Output in JSON format

#### bd ready

Show ready work (no blockers).

```bash
# Show all ready work
bd ready

# Limit results
bd ready --limit 20

# Filter by priority
bd ready --priority 1

# Filter by assignee
bd ready --assignee alice

# JSON for agent scripts
bd ready --json | jq '.[0]'
```

**Flags:**
- `--limit INT` - Max number of results
- `--priority INT` - Filter by priority
- `--assignee USER` - Filter by assignee
- `--json` - Output in JSON format

**Agent pattern:**
```bash
WORK=$(bd ready --limit 1 --json)
ISSUE_ID=$(echo "$WORK" | jq -r '.[0].id')
bd update "$ISSUE_ID" --status in_progress --json
```

#### bd blocked

Show blocked issues.

```bash
bd blocked
bd blocked --json
```

Shows issues that have open blockers or are blocked by parent hierarchy.

#### bd stats

Show statistics.

```bash
bd stats
bd stats --json
```

Shows counts by status, priority, type, etc.

### Updating Issues

#### bd update

Update issue fields.

```bash
# Update status
bd update bd-1 --status in_progress
bd update bd-1 --status open

# Update priority
bd update bd-1 --priority 0

# Update assignee
bd update bd-1 --assignee bob

# Update type
bd update bd-1 --type bug

# Update multiple fields
bd update bd-1 --status in_progress --priority 1 --assignee alice --json

# Update description, design, notes, acceptance criteria
bd update bd-1 --description "New description"
bd update bd-1 --design "Design details"
bd update bd-1 --notes "Implementation notes"
bd update bd-1 --acceptance-criteria "Must pass all tests"
```

**Flags:**
- `--status STATUS` - Status: open, in_progress, closed, blocked
- `--priority INT` - Priority: 0-4
- `--assignee USER` - Assignee
- `--type TYPE` - Type: bug, feature, task, epic, chore
- `--description TEXT` - Description
- `--design TEXT` - Design notes
- `--notes TEXT` - Implementation notes
- `--acceptance-criteria TEXT` - Acceptance criteria
- `--estimated-minutes INT` - Time estimate
- `--actual-minutes INT` - Actual time spent
- `--labels LIST` - Comma-separated labels (replaces existing)
- `--json` - Output in JSON format

**Status transitions:**
- `open` → `in_progress` (claiming work)
- `in_progress` → `closed` (completing work)
- `in_progress` → `open` (releasing work)
- `closed` → `open` (reopening)
- Any → `blocked` (manual block, though usually automatic via dependencies)

#### bd close

Close issues with reason.

```bash
# Close single issue
bd close bd-1 --reason "Completed"
bd close bd-1 --reason "Fixed" --json

# Close multiple issues
bd close bd-1 bd-2 bd-3 --reason "Batch completion"
```

**Flags:**
- `--reason TEXT` - Closure reason (required)
- `--json` - Output in JSON format

**Best practice:** Always provide descriptive reason (not just "done").

### Dependencies

#### bd dep add

Add dependency between issues.

```bash
# Blocking dependency (default)
bd dep add bd-5 bd-3              # bd-5 depends on bd-3
bd dep add bd-5 bd-3 --type blocks

# Related (soft relationship)
bd dep add bd-10 bd-8 --type related

# Parent-child (epic hierarchy)
bd dep add bd-15 bd-12 --type parent-child  # bd-15 child of bd-12

# Discovered-from (track discovery)
bd dep add bd-21 bd-20 --type discovered-from  # bd-21 discovered from bd-20

# Or use --deps flag during creation
bd create "Fix bug" --deps discovered-from:bd-20
```

**Syntax:** `bd dep add <from-issue> <to-issue> [--type TYPE]`

**Flags:**
- `--type TYPE` - Dependency type: blocks, related, parent-child, discovered-from (default: blocks)

**Remember:**
- Only `blocks` affects ready work
- Hierarchical blocking propagates through `parent-child`
- `related` and `discovered-from` are for context only

#### bd dep remove

Remove dependency.

```bash
bd dep remove bd-5 bd-3
bd dep remove bd-10 bd-8
```

**Syntax:** `bd dep remove <from-issue> <to-issue>`

Removes the dependency regardless of type.

#### bd dep tree

Visualize dependency tree.

```bash
bd dep tree bd-2
bd dep tree bd-2 --json
```

**Flags:**
- `--json` - Output in JSON format

Shows:
- Issue and all dependencies
- Dependency types
- Blocker chain
- Hierarchical structure

**Best practice:** Use before starting work to understand full dependency chain.

#### bd dep cycles

Detect dependency cycles.

```bash
bd dep cycles
bd dep cycles --json
```

**Flags:**
- `--json` - Output in JSON format

Finds circular dependencies that would cause deadlock. bd prevents cycles during `dep add`, but this command verifies database integrity.

### Export and Import

#### bd export

Export issues to JSONL.

```bash
# Export all issues
bd export --format=jsonl
bd export --format=jsonl -o issues.jsonl

# Export filtered
bd export --format=jsonl --status=open -o open.jsonl
bd export --format=jsonl --priority=0 -o critical.jsonl
```

**Flags:**
- `--format FORMAT` - Format: jsonl (default)
- `-o, --output FILE` - Output file (default: stdout)
- `--status STATUS` - Filter by status
- `--priority INT` - Filter by priority
- `--assignee USER` - Filter by assignee
- `--type TYPE` - Filter by type

**JSONL format:** One JSON object per line, sorted by ID for consistent diffs.

**Best practice:** Automatic export happens after CRUD ops. Manual export useful for backups or filtered exports.

#### bd import

Import issues from JSONL.

```bash
# Import from file
bd import -i issues.jsonl

# Import from stdin
cat issues.jsonl | bd import

# Skip existing (only create new)
bd import -i issues.jsonl --skip-existing

# Preview collisions without changes
bd import -i issues.jsonl --dry-run

# Auto-resolve collisions
bd import -i issues.jsonl --resolve-collisions
```

**Flags:**
- `-i, --input FILE` - Input file (default: stdin)
- `--skip-existing` - Only create new issues, skip existing IDs
- `--dry-run` - Preview changes without applying
- `--resolve-collisions` - Automatically renumber colliding issues

**Import behavior:**
- Existing issues (same ID) are **updated** with new values
- New issues are **created**
- All imports are atomic (all or nothing)

**Collision resolution:**

When importing issues with IDs that exist but have different content:

1. **Detect collisions:**
```bash
bd import -i branch-issues.jsonl --dry-run
# Shows: exact matches, new issues, collisions with conflicting fields
```

2. **Auto-resolve:**
```bash
bd import -i branch-issues.jsonl --resolve-collisions
# Renumbers colliding issues to new IDs
# Updates all text references and dependencies
# Reports remapping (bd-10 → bd-25, etc.)
```

**Algorithm:**
- Keeps existing database issues unchanged
- Assigns new IDs to colliding incoming issues
- Updates ALL references (text mentions, dependencies)
- Prioritizes by reference count (fewest refs remapped first)
- Uses word-boundary matching (bd-10 ≠ bd-100)

**Best practice:** Always use `--dry-run` first when merging branches.

## Agent Workflow Patterns

### Standard Workflow

```bash
# 1. Find ready work
WORK=$(bd ready --limit 1 --json)
ISSUE_ID=$(echo "$WORK" | jq -r '.[0].id')

# 2. Claim work
bd update "$ISSUE_ID" --status in_progress --json

# 3. While working, discover new issue
NEW_ISSUE=$(bd create "Fix edge case bug" -t bug -p 1 \
  --deps discovered-from:"$ISSUE_ID" --json)
NEW_ID=$(echo "$NEW_ISSUE" | jq -r '.id')

# 4. Complete work
bd close "$ISSUE_ID" --reason "Implemented and tested" --json

# 5. Find next ready work (discovered issue might be ready now)
bd ready --json
```

### Epic with Subtasks

```bash
# 1. Create epic
EPIC=$(bd create "Epic: User Authentication" -t epic -p 1 --json)
EPIC_ID=$(echo "$EPIC" | jq -r '.id')

# 2. Create subtasks
for task in "Add login form" "Add signup form" "Add password reset"; do
  SUBTASK=$(bd create "$task" -t task -p 1 --json)
  SUBTASK_ID=$(echo "$SUBTASK" | jq -r '.id')
  bd dep add "$SUBTASK_ID" "$EPIC_ID" --type parent-child
done

# 3. Find ready subtasks
bd ready --json | jq 'map(select(.title | contains("login")))'
```

### Blocking Chain

```bash
# Create chain: Design → Implement → Test → Deploy
DESIGN=$(bd create "Design API" -t task -p 0 --json | jq -r '.id')
IMPL=$(bd create "Implement API" -t task -p 1 --json | jq -r '.id')
TEST=$(bd create "Test API" -t task -p 1 --json | jq -r '.id')
DEPLOY=$(bd create "Deploy API" -t task -p 2 --json | jq -r '.id')

# Set up blocking chain
bd dep add "$IMPL" "$DESIGN" --type blocks
bd dep add "$TEST" "$IMPL" --type blocks
bd dep add "$DEPLOY" "$TEST" --type blocks

# Only design is ready
bd ready --json
# [{"id": "bd-1", "title": "Design API", ...}]
```

### Session Recovery

```bash
# Agent boots up, no context
# Find where we left off
bd list --status in_progress --json

# Find what's ready to work on
bd ready --limit 5 --json

# Check if blocked
bd blocked --json

# Get full context on an issue
bd show bd-42 --json
bd dep tree bd-42
```

### Multi-Project Handling

```bash
# Project 1
cd ~/project1
bd init --prefix proj1
bd create "Task for project 1"

# Project 2
cd ~/project2
bd init --prefix proj2
bd create "Task for project 2"

# Commands auto-discover correct database
cd ~/project1/src/components
bd list  # Uses ~/project1/.beads/proj1.db

cd ~/project2/tests
bd list  # Uses ~/project2/.beads/proj2.db
```

## Quick Reference

### All Commands

| Command | Purpose |
|---------|---------|
| `bd init` | Initialize Beads in project |
| `bd create` | Create new issue |
| `bd show` | Show issue details |
| `bd list` | List issues with filters |
| `bd ready` | Show unblocked work |
| `bd blocked` | Show blocked issues |
| `bd stats` | Show statistics |
| `bd update` | Update issue fields |
| `bd close` | Close issue with reason |
| `bd dep add` | Add dependency |
| `bd dep remove` | Remove dependency |
| `bd dep tree` | Visualize dependency tree |
| `bd dep cycles` | Detect circular dependencies |
| `bd export` | Export to JSONL |
| `bd import` | Import from JSONL |
| `bd quickstart` | Interactive tutorial |
| `bd version` | Show version |
| `bd --help` | Show help |

### Dependency Type Decision Matrix

| Situation | Type | Command |
|-----------|------|---------|
| X cannot start until Y completes | blocks | `bd dep add X Y --type blocks` |
| X and Y are related but independent | related | `bd dep add X Y --type related` |
| X is subtask of epic Y | parent-child | `bd dep add X Y --type parent-child` |
| Discovered X while working on Y | discovered-from | `bd create "X" --deps discovered-from:Y` |

### Status Transitions

| From | To | When |
|------|-----|------|
| open | in_progress | Claiming work |
| in_progress | closed | Completing work |
| in_progress | open | Releasing unclaimed |
| closed | open | Reopening |
| * | blocked | Manual (or automatic via dependencies) |

### Priority Levels

| Priority | Name | Use For |
|----------|------|---------|
| 0 | Critical | Must fix now, blocking everything |
| 1 | High | Important, work on soon |
| 2 | Normal | Standard priority (default) |
| 3 | Low | Nice to have |
| 4 | Lowest | Backlog |

### Issue Types

| Type | Use For |
|------|---------|
| bug | Defects, broken behavior |
| feature | New functionality |
| task | General work item |
| epic | Large initiative with subtasks |
| chore | Maintenance, refactoring |

## Common Mistakes

### Missing --json Flags

**Wrong:**
```bash
ISSUE=$(bd create "Task")
ID=$(echo "$ISSUE" | grep -o 'bd-[0-9]*')  # Fragile parsing
```

**Right:**
```bash
ISSUE=$(bd create "Task" --json)
ID=$(echo "$ISSUE" | jq -r '.id')  # Reliable
```

**Always use `--json` in agent scripts for parsing.**

### Wrong Dependency Type

**Wrong:**
```bash
# Using blocks when you mean related
bd dep add bd-5 bd-3 --type blocks  # Now bd-5 won't be ready until bd-3 closes
```

**Right:**
```bash
# Use related for context-only relationships
bd dep add bd-5 bd-3 --type related  # bd-5 can still be worked on
```

**Remember:** Only `blocks` affects ready work. Use `related` for context.

### Forgetting Hierarchical Blocking

**Wrong:**
```bash
# Thinking child task is ready because it has no direct blockers
bd create "Epic" -t epic  # bd-1
bd create "Subtask" -t task  # bd-2
bd dep add bd-2 bd-1 --type parent-child
bd create "Blocker" -t task  # bd-3
bd dep add bd-1 bd-3 --type blocks  # Epic blocked

bd ready  # Expecting to see bd-2... but it's blocked via parent!
```

**Right:**
```bash
# Check blocked to see hierarchical blocking
bd blocked  # Shows both bd-1 and bd-2
bd dep tree bd-2  # Shows parent is blocked
```

**Remember:** Children inherit parent's blocked status.

### Not Linking Discovered Work

**Wrong:**
```bash
# While working on bd-20, discover bug
bd create "Fix bug"  # No link back to bd-20
# Lost context of where this came from
```

**Right:**
```bash
# Link back to discovery source
bd create "Fix bug" --deps discovered-from:bd-20
# Or separate commands:
BUG=$(bd create "Fix bug" --json | jq -r '.id')
bd dep add "$BUG" bd-20 --type discovered-from
```

**Always link discovered work back to source.**

### Import Without Dry-Run

**Wrong:**
```bash
git pull
bd import -i .beads/issues.jsonl --resolve-collisions  # Blindly remapping
```

**Right:**
```bash
git pull
bd import -i .beads/issues.jsonl --dry-run  # Preview first
# Review collisions
bd import -i .beads/issues.jsonl --resolve-collisions  # Then apply
```

**Always preview with `--dry-run` when merging branches.**

### Skipping bd ready

**Wrong:**
```bash
# Just grab first open issue
bd list --status open --json | jq '.[0]'
# Might be blocked!
```

**Right:**
```bash
# Use ready to get unblocked work
bd ready --json | jq '.[0]'
# Guaranteed to have no blockers
```

**Always use `bd ready` to find work, not `bd list --status open`.**

### Ignoring Cycle Detection

**Wrong:**
```bash
bd dep add bd-1 bd-2 --type blocks
bd dep add bd-2 bd-3 --type blocks
bd dep add bd-3 bd-1 --type blocks  # Creates cycle!
# bd prevents this, but if somehow happens...
bd ready  # Shows nothing (everything blocked)
```

**Right:**
```bash
# Run cycle detection
bd dep cycles
# Fix any cycles found
bd dep remove bd-3 bd-1
```

**Run `bd dep cycles` if ready work is empty unexpectedly.**

### Vague Titles/Descriptions

**Wrong:**
```bash
bd create "Fix bug"  # Which bug?
bd create "Update code"  # Which code?
bd close bd-1 --reason "Done"  # What was done?
```

**Right:**
```bash
bd create "Fix login validation accepting empty emails" -t bug
bd create "Update UserService to hash passwords with bcrypt" -t task
bd close bd-1 --reason "Implemented email validation with regex, added tests"
```

**Be specific. Future agents need context.**

## Integration Patterns

### Error Handling

```bash
# Check if bd is available
if ! command -v bd &> /dev/null; then
  echo "Error: bd not installed"
  exit 1
fi

# Check if initialized
if ! bd list &> /dev/null; then
  echo "Error: bd not initialized. Run: bd init"
  exit 1
fi

# Handle no ready work
WORK=$(bd ready --json)
if [ "$WORK" = "[]" ]; then
  echo "No ready work available"
  exit 0
fi
```

### JSON Parsing

```bash
# Extract fields
ISSUE=$(bd show bd-1 --json)
TITLE=$(echo "$ISSUE" | jq -r '.title')
STATUS=$(echo "$ISSUE" | jq -r '.status')
PRIORITY=$(echo "$ISSUE" | jq -r '.priority')

# Filter ready work
bd ready --json | jq 'map(select(.priority <= 1))'  # P0 and P1 only
bd ready --json | jq 'map(select(.type == "bug"))'  # Bugs only

# Extract first ready issue
NEXT=$(bd ready --limit 1 --json | jq -r '.[0].id')
```

### Batch Operations

```bash
# Close multiple issues
for issue in bd-1 bd-2 bd-3; do
  bd close "$issue" --reason "Batch completion" --json
done

# Update all bugs to high priority
bd list --type bug --json | jq -r '.[].id' | while read id; do
  bd update "$id" --priority 1
done

# Find and block all issues needing design
bd list --label needs-design --json | jq -r '.[].id' | while read id; do
  bd update "$id" --status blocked
done
```

### Git Integration

```bash
# Pre-commit hook: Export before commit
#!/bin/bash
bd export -o .beads/issues.jsonl
git add .beads/issues.jsonl

# Post-merge hook: Import after merge
#!/bin/bash
bd import -i .beads/issues.jsonl
```

## When NOT to Use Beads

**Don't use Beads for:**
- Single-file scripts - Too simple, no dependencies
- Throwaway prototypes - Won't be maintained
- Projects completed in single session - No multi-session memory needed
- Already using another issue tracker - Don't duplicate

**Use project's existing tools if available.** Beads is for projects that need dependency-aware tracking and don't have it.

## Troubleshooting

### "bd: command not found"

Not installed. Install from: https://github.com/steveyegge/beads

```bash
# Using go
go install github.com/steveyegge/beads/cmd/bd@latest

# Or brew
brew tap steveyegge/beads && brew install bd
```

### "database is locked"

Another bd process is running or didn't close cleanly.

```bash
# Kill hanging processes
ps aux | grep bd
kill <pid>

# Remove lock files
rm .beads/*.db-journal .beads/*.db-wal .beads/*.db-shm
```

### Ready work empty but have open issues

Issues are blocked. Check:

```bash
bd blocked  # See what's blocked
bd dep tree bd-1  # Check blocking chain
bd dep cycles  # Check for circular dependencies
```

### Import fails with collisions

Use `--dry-run` to preview, then `--resolve-collisions`:

```bash
bd import -i issues.jsonl --dry-run
bd import -i issues.jsonl --resolve-collisions
```

## Advanced Topics

For complete documentation, see:
- `bd quickstart` - Interactive tutorial
- `bd --help` - Full command reference
- `bd <command> --help` - Command-specific help
- https://github.com/steveyegge/beads - Complete README

## Summary

**Before starting ANY multi-session work:**
1. Check if bd is initialized: `bd list`
2. If not: `bd init`

**Standard agent workflow:**
1. Find ready work: `bd ready --json`
2. Claim work: `bd update <id> --status in_progress`
3. Create discovered issues: `bd create --deps discovered-from:<id>`
4. Complete work: `bd close <id> --reason "..."`
5. Repeat

**Critical rules:**
- Always use `--json` in agent scripts
- Use `bd ready`, not `bd list --status open`
- Link discovered work with `discovered-from`
- Only `blocks` affects ready work
- Children inherit parent blocking
- Preview imports with `--dry-run`

**When stuck:** Run `bd quickstart` for interactive guidance.

