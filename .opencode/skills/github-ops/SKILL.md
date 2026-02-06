---
name: github-ops
description: GitHub CLI operations for issue tracking, commits, and repository management
---

# GitHub Operations Skill

This skill provides patterns for GitHub CLI (gh) operations in OSPM workflows.

## Core Patterns

### Get Repository

```bash
get_repo() {
  local remote_url=$(git remote get-url origin 2>/dev/null || echo "")
  local repo=$(echo "$remote_url" | sed 's|.*github.com[:/]||' | sed 's|\.git$||')
  [ -z "$repo" ] && repo="user/repo"
  echo "$repo"
}
REPO=$(get_repo)
```

### Get Current DateTime

```bash
CURRENT_DATE=$(date -u +"%Y-%m-%dT%H:%M:%SZ")
```

## Issue Operations

### Create Issue

```bash
gh issue create \
  --repo "$REPO" \
  --title "Title here" \
  --body "@FILE:body.md" \
  --label "label1,label2"
```

### Create Issue from File

```bash
# First, prepare the body file
cat > /tmp/issue-body.md << 'EOF'
Issue body content here
EOF

# Then create
gh issue create \
  --repo "$REPO" \
  --title "Issue Title" \
  --body "@/tmp/issue-body.md" \
  --label "task"
```

### View Issue Details

```bash
# Get all info as JSON
gh issue view 123 --json state,title,labels,body,assignees

# Get specific fields
gh issue view 123 --json state,title -q '.state + " " + .title'

# Just the number
gh issue view 123 --json number -q .number
```

### List Issues

```bash
# All open issues
gh issue list --repo "$REPO"

# Filtered by label
gh issue list --repo "$REPO" --label "epic"
gh issue list --repo "$REPO" --label "task"

# JSON output
gh issue list --repo "$REPO" --json number,title,state,labels
```

### Edit Issue

```bash
# Add label
gh issue edit 123 --add-label "status:in-progress"

# Remove label
gh issue edit 123 --remove-label "status:open"

# Add assignee
gh issue edit 123 --add-assignee @me

# Set title
gh issue edit 123 --title "New Title"

# Set body
gh issue edit 123 --body "@FILE:body.md"
```

### Close Issue

```bash
gh issue close 123 --comment "Closing - implementation complete"
```

### Reopen Issue

```bash
gh issue reopen 123 --comment "Reopening for additional work"
```

### Comment on Issue

```bash
# From file
gh issue comment 123 --body "@FILE:comment.md"

# Inline
gh issue comment 123 --body "Status update: Implementation in progress"
```

## Label Operations

### Create Label

```bash
gh label create "epic" --repo "$REPO" --description "Epic-level issue"
gh label create "task" --repo "$REPO" --description "Individual task"
gh label create "feature-name" --repo "$REPO" --description "Feature scope"
```

### List Labels

```bash
gh label list --repo "$REPO"
```

## Commit Operations

### Get Current Branch

```bash
BRANCH=$(git branch --show-current)
```

### Get Latest Commit

```bash
# Short hash
COMMIT_HASH=$(git rev-parse --short HEAD)

# Full hash
COMMIT_HASH=$(git rev-parse HEAD)

# Message
COMMIT_MSG=$(git log -1 --pretty=%B)
```

### Create Commit

```bash
git add .
git commit -m "Task #123: Implement feature description

- Changes made
- Testing approach
- Related: #456"
```

## Worktree Operations

### Create Worktree

```bash
# Ensure main is current
git checkout main
git pull origin main

# Create worktree
git worktree add ../epic-feature-name -b epic/feature-name
```

### List Worktrees

```bash
git worktree list
```

### Remove Worktree

```bash
git worktree remove ../epic-feature-name
```

## Repository Operations

### View Repository

```bash
# Basic info
gh repo view

# With owner
gh repo view --json nameWithOwner -q .nameWithOwner

# Default branch
gh repo view --json defaultBranchRef -q .defaultBranchRef.name
```

### Check Authentication

```bash
gh auth status
gh auth login
```

## Useful Patterns

### Extract Issue Number from Branch

```bash
# From branch epic/user-auth
EPIC_NAME=$(git branch --show-current | sed 's/epic\///')

# From commit message "Task #123:"
TASK_NUM=$(git log -1 --grep="Task #" -Eo "Task #[0-9]+" | grep -o "[0-9]*")
```

### Get Issue URL

```bash
REPO=$(gh repo view --json nameWithOwner -q .nameWithOwner)
ISSUE_URL="https://github.com/$REPO/issues/$1"
```

### Update Issue from Task File

```bash
# Extract and strip frontmatter
sed '1,/^---$/d; 1,/^---$/d' task-123.md > /tmp/task-body.md

# Update issue
gh issue edit 123 --body "@/tmp/task-body.md"
```

## Error Handling

### Handle gh Errors

```bash
# Check if gh is available
command -v gh >/dev/null 2>&1 || { echo "❌ GitHub CLI not installed"; exit 1; }

# Check auth
gh auth status >/dev/null 2>&1 || { echo "❌ Not authenticated with GitHub"; exit 1; }

# Check repo
gh repo view "$REPO" >/dev/null 2>&1 || { echo "❌ Repository not accessible"; exit 1; }

# Handle command failure
gh issue create ... || { echo "❌ Failed to create issue"; exit 1; }
```

### Retry Pattern

```bash
# Retry up to 3 times
for i in {1..3}; do
  gh issue create --repo "$REPO" ... && break
  sleep 2
done
```

## Best Practices

1. **Always specify `--repo`**: Don't rely on gh context
2. **Use `--json` for parsing**: Structured output is reliable
3. **Prefer `--body-file`**: Handles special characters better
4. **Check authentication first**: Fail fast on auth issues
5. **Use labels consistently**: Same labels across issues
6. **Link commits to issues**: Use `#123` in commit messages
7. **Comment on progress**: Keep issue comments updated
8. **Close with context**: Explain why and link PRs

## Common Labels

| Label | Color | Purpose |
|-------|-------|---------|
| epic | purple | Parent tracking issue |
| task | blue | Individual work item |
| bug | red | Bug fix work |
| feature | green | New feature work |
| enhancement | yellow | Improvement work |
| status:open | grey | Not started |
| status:in-progress | orange | Active work |
| status:blocked | red | Waiting on dependencies |
| status:done | green | Complete |
