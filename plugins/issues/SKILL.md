---
name: issues
description: "GitHub Issues task tracker for any repo. Manages tasks as GitHub issues in the current repo. Use when: tracking TODOs, creating tasks, listing tasks, showing task status, updating tasks, marking tasks done, planning multi-step work, managing a backlog, asking what to work on next, or when the user mentions anything about work items, action items, or things to do later. Persists across sessions and lives in the repo's issue tracker."
allowed-tools: Bash, Read, Write, Edit, Glob
---

# Issues — GitHub Issues Task Tracker

Tasks are **GitHub issues** in the current repo. There is no `.tasks/` directory and no local files. The repo is auto-detected from `git remote get-url origin` (must point to github.com).

## Access Check
!`url=$(git remote get-url origin 2>/dev/null); case "$url" in *github.com[:/]*) if gh auth status >/dev/null 2>&1 || [ -n "${GH_PAT:-}${GH_TOKEN:-}${GITHUB_TOKEN:-}" ]; then echo "OK"; else echo "NO_AUTH — run 'gh auth login' or export GH_PAT=<token> (read/write Issues)"; fi ;; *) echo "NO_REPO — run inside a git repo whose origin points to github.com" ;; esac`

## Current Tasks
!`gh issue list --state open --limit 50 2>/dev/null || echo "No issues yet (or gh not authenticated)."`

---

This skill uses the **`gh` CLI**. Make sure `gh auth status` succeeds (or a `$GH_PAT`/`$GH_TOKEN` with read/write Issues access is exported — `gh` reads `GH_TOKEN` automatically). If the Access Check above is `NO_REPO` or `NO_AUTH`, stop and tell the user how to fix it.

## How tasks map to GitHub issues

| Task concept | GitHub issue                       |
|--------------|------------------------------------|
| id           | issue number (e.g. `#42`)          |
| name         | issue title                        |
| description  | issue body (the plan lives here)   |
| pending      | open, no status label              |
| in_progress  | open + `in_progress` label         |
| done         | closed                             |
| priority 0–3 | `priority:0`..`priority:3` label   |
| later        | no priority label (default)        |

## CRITICAL: Context-Efficient Workflow

**Always start with `gh issue list` to read titles/labels only.** Never fetch full issue bodies unless the user asks for details on a specific task. This keeps context small.

## When to Use This Skill (Be Proactive!)

Proactively suggest creating issues when:
- The user describes multi-step work ("I need to add auth, then set up roles, then...")
- The user mentions something they'll do later ("I'll fix that tomorrow", "we should also...")
- You discover TODO/FIXME/HACK comments in code during your work
- A bug is found but not immediately fixable
- The user asks "what should I work on?" — list their open issues

## Commands

Parse the user's `$ARGUMENTS` to determine the action:

### `list` (default when no argument given)

```bash
gh issue list --state open
```

Format as a markdown table (number, title, labels). If none: "No open issues. Want me to create some to track your work?"

### `show <number>`

```bash
gh issue view <number>
```

Render the body as markdown and show state + labels.

### `create`

Create an issue. GitHub assigns the number — you do not pick IDs.

```bash
gh issue create --title "Concise task title" --body "Full plan details"
```

- **title**: concise summary (required)
- **body**: full plan details — the whole point of a task
- To set priority, add a label (create it first if missing):
  ```bash
  gh label create "priority:1" --force
  gh issue create --title "..." --body "..." --label "priority:1"
  ```

### `update <number>`

```bash
# Title / body
gh issue edit <number> --title "New title"
gh issue edit <number> --body "New body"

# Mark in progress
gh label create "in_progress" --force && gh issue edit <number> --add-label "in_progress"

# Set priority (swap labels)
gh issue edit <number> --remove-label "priority:2" --add-label "priority:1"
```

### `done <number>` / `close <number>`

"Done" closes the issue (not deleted — reopenable). Confirm with the user first.

```bash
gh issue close <number>
```

## Priority & Status

- Priority labels: `priority:0` (drop everything) → `priority:1` (high) → `priority:2` (normal) → `priority:3` (low). No label = `later`.
- Status: open = active; `in_progress` label = being worked on; closed = done.

## Workflow Integration

When working on a task:
1. **Start**: `gh issue edit <number> --add-label in_progress` before beginning work
2. **Progress**: update the issue body with notes as you go (files changed, decisions made)
3. **Complete**: `gh issue close <number>` when done
4. **Next**: show remaining open issues and ask what to tackle next

## Rules

- **Never fetch full issue bodies during list** — titles/labels only
- **Numbers are assigned by GitHub** — never invent IDs
- **Bodies should be thorough** — include file paths, line numbers, step-by-step plans
- If no argument is provided, default to `list`
- Each repo has its own issues — tasks never leak across repos
- **Be proactive** — suggest creating issues when you see multi-step work ahead
- "Done" = close (preserves history); never delete issues
