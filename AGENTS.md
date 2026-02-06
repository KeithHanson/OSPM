# OSPM: OpenCode Simple PM

A lightweight, GitHub-native project management system for AI-assisted development.

## Overview

OSPM provides a structured workflow from idea to shipped code:

```
PRD (local) → Epic (GitHub Issue) → Tasks (GitHub Issues) → Worktree → Commit → Ship
```

### Core Principles

1. **GitHub as Source of Truth**: Issues track work, commits document changes
2. **Minimal Local State**: Only PRDs stored locally; everything else in GitHub
3. **Concurrency Analysis**: Identify parallelizable work for faster delivery
4. **Full Traceability**: Every commit, PR, and issue connects back to requirements

## OSPM Workflow

### Phase 1: Create PRD
- Write PRD in `prds/` directory following the PRD template
- Commit PRD to git with descriptive message

### Phase 2: Create Epic Issue
- Create GitHub issue with `epic` label
- Link to PRD commit in issue body

### Phase 3: Create Task Issues
- Break epic into concrete, actionable tasks
- Create each task as GitHub issue with `task` label
- Use task lists for parent-child relationships (no gh-sub-issue extension needed)
- Analyze concurrency: mark parallelizable tasks

### Phase 4: Execute Work
- Create worktree for the epic: `git worktree add ../epic-name -b epic/name`
- Pull latest issues to understand current state
- Work on tasks in parallel when possible
- Commit with task reference: `git commit -m "Task #123: Description"`

### Phase 5: Ship
- Merge worktree branch to main
- Close epic and task issues
- Link PR in issue comments

---

## PRD Creation

### When to Create a PRD

Create a PRD for:
- New features requiring architectural decisions
- Complex changes affecting multiple components
- External integrations or API changes
- Any work needing upfront planning

### PRD Template

```markdown
---
name: feature-name
description: Brief one-line description
status: backlog
created: 2024-01-15T14:30:45Z
---

# PRD: feature-name

## Executive Summary
Brief overview and value proposition

## Problem Statement
What problem are we solving? Why is this important now?

## User Stories
- As a [user type], I want [goal] so that [benefit]

## Requirements

### Functional Requirements
- Core feature 1
- Core feature 2

### Non-Functional Requirements
- Performance: [expectations]
- Security: [considerations]
- Scalability: [needs]

## Success Criteria
- Measurable outcome 1
- Measurable outcome 2

## Constraints & Assumptions
- Technical limitations
- Timeline constraints

## Out of Scope
- What we're NOT building

## Dependencies
- External dependencies
- Internal dependencies
```

### PRD Creation Steps

1. **Get current datetime**:
   ```bash
   CURRENT_DATE=$(date -u +"%Y-%m-%dT%H:%M:%SZ")
   ```

2. **Create file**: `prds/feature-name.md`

3. **Fill in template** with real content (no placeholders)

4. **Validate**: Ensure all sections are complete with actual content

5. **Commit**:
   ```bash
   git add prds/feature-name.md
   git commit -m "PRD: Add feature-name - Brief description of feature"
   ```

---

## Epic Creation

### Epic Issue Template

Title: `Epic: feature-name`

Body:
```markdown
## Overview
Brief technical summary of the implementation approach

## Architecture Decisions
- Decision 1 with rationale
- Decision 2 with rationale

## Technical Approach
### Frontend Components
- Component 1
- Component 2

### Backend Services
- API endpoint 1
- API endpoint 2

## Implementation Strategy
- Development phases
- Risk mitigation

## Task Breakdown
- [ ] #TASK_NUMBER - Task title (parallel: true/false)

## Dependencies
- External dependencies
- Internal dependencies

## Success Criteria
- Technical benchmarks
- Quality gates

## Links
- PRD: [commit hash](link to commit or file)
```

### Create Epic Issue

```bash
# Get repository
REPO=$(gh repo view --json nameWithOwner -q .nameWithOwner)

# Create epic issue
gh issue create \
  --repo "$REPO" \
  --title "Epic: feature-name" \
  --body "@FILE:epic-body.md" \
  --label "epic,feature-name"
```

---

## Task Creation

### Task File Template

Each task should have:

```markdown
---
name: task-title
status: open
created: 2024-01-15T14:30:45Z
updated: 2024-01-15T14:30:45Z
github: https://github.com/owner/repo/issues/123
depends_on: []  # List task numbers: [123, 124]
parallel: true  # Can run in parallel with other tasks
conflicts_with: []  # Tasks modifying same files: [125, 126]
---

# Task: task-title

## Description
Clear description of what needs to be done

## Acceptance Criteria
- [ ] Criterion 1
- [ ] Criterion 2
- [ ] Criterion 3

## Technical Details
- Implementation approach
- Files affected
- Key considerations

## Dependencies
- Task dependencies: [number]
- External dependencies

## Effort Estimate
- Size: XS/S/M/L/XL
- Hours: estimated hours
```

### Task Fields Explained

| Field | Purpose |
|-------|---------|
| `depends_on` | Tasks that must complete before this can start |
| `parallel` | Can run simultaneously with other tasks |
| `conflicts_with` | Tasks modifying the same files (risk of merge conflicts) |

### Concurrency Analysis

When creating tasks, identify:

1. **Parallelizable tasks**: Can run at the same time
   - Different components (UI, API, database)
   - Different features (authentication, search, notifications)
   - Set `parallel: true`

2. **Dependent tasks**: Must run sequentially
   - Database before API integration
   - Core logic before UI
   - Set `depends_on: [task_numbers]`

3. **Conflicting tasks**: Risk of merge conflicts
   - Same file modifications
   - Shared configuration changes
   - Set `conflicts_with: [task_numbers]`

### Create Task Issues

```bash
# Get repository
REPO=$(gh repo view --json nameWithOwner -q .nameWithOwner)

# Create task issue
gh issue create \
  --repo "$REPO" \
  --title "Task: task-title" \
  --body "@FILE:task-body.md" \
  --label "task,feature-name"
```

### Task List in Epic

Add task list to epic issue body:

```markdown
## Tasks
- [ ] #123 - Task 1 description (parallel: true)
- [ ] #124 - Task 2 description (parallel: true)
- [ ] #125 - Task 3 description (parallel: false, depends on #123)
```

---

## GitHub Operations

### Repository Detection

```bash
get_repo() {
  local remote_url=$(git remote get-url origin 2>/dev/null || echo "")
  local repo=$(echo "$remote_url" | sed 's|.*github.com[:/]||' | sed 's|\.git$||')
  [ -z "$repo" ] && repo="user/repo"
  echo "$repo"
}
```

### Issue Operations

#### Create Issue
```bash
gh issue create \
  --repo "$REPO" \
  --title "Title" \
  --body "@FILE:body.md" \
  --label "label1,label2"
```

#### View Issue
```bash
gh issue view 123 --json state,title,labels,body
```

#### Edit Issue
```bash
gh issue edit 123 --add-label "status" --add-assignee @me
```

#### Comment on Issue
```bash
gh issue comment 123 --body "@FILE:comment.md"
```

#### Close Issue
```bash
gh issue close 123 --comment "Closing with resolution"
```

#### Reopen Issue
```bash
gh issue reopen 123 --comment "Reopening for additional work"
```

### Label Management

| Label | Purpose |
|-------|---------|
| `epic` | Marks epic-level issues |
| `task` | Marks individual task issues |
| `feature-name` | Scope-specific labeling |
| `status:in-progress` | Work actively being done |
| `status:blocked` | Blocked on dependencies |
| `status:done` | Completed |

### Authentication

```bash
# Check authentication
gh auth status

# If not authenticated
gh auth login
```

---

## Worktree Conventions

### Create Worktree

```bash
# Ensure main is current
git checkout main
git pull origin main

# Create worktree for epic
git worktree add ../epic-feature-name -b epic/feature-name

# Navigate to worktree
cd ../epic-feature-name
```

### Worktree Naming

```
../epic-{feature-name}
Branch: epic/{feature-name}
```

### Worktree Workflow

```bash
# 1. Create worktree
git worktree add ../epic-auth -b epic/user-authentication

# 2. Work in parallel (multiple worktrees possible)
cd ../epic-auth
# Make changes
git add .
git commit -m "Task #123: Implement user authentication flow"

# 3. When done, merge and cleanup
git checkout main
git merge epic/user-authentication --ff-only
git worktree remove ../epic-auth
```

### Parallel Worktrees

For maximum parallelism, create separate worktrees for independent tasks:

```bash
# Task 1: Database
git worktree add ../epic-db -b epic/database-layer

# Task 2: API
git worktree add ../epic-api -b epic/api-endpoints

# Task 3: UI
git worktree add ../epic-ui -b epic/user-interface

# All can work simultaneously!
```

---

## Specialized Agents

### code-analyzer

**Purpose**: Hunt bugs across multiple files without polluting main context

**Pattern**: Search → Analyze → Return bug report

**Usage**:
```bash
Task: "Search for memory leaks in the codebase"
Agent: code-analyzer
Returns: "Found 3 potential leaks: [concise list]"
```

**Returns**: Concise bug report with critical findings only

### file-analyzer

**Purpose**: Read and summarize verbose files (logs, outputs, configs)

**Pattern**: Read → Extract → Return summary

**Usage**:
```bash
Task: "Summarize test output"
Agent: file-analyzer
Returns: "2/10 tests failed: [failure summary]"
```

**Returns**: Key findings and actionable insights

### test-runner

**Purpose**: Execute tests without dumping output to main thread

**Pattern**: Run → Capture → Analyze → Return summary

**Usage**:
```bash
Task: "Run authentication tests"
Agent: test-runner
Returns: "All tests passing" or "2 failures: [details]"
```

**Returns**: Test results with failure analysis

### parallel-worker

**Purpose**: Coordinate multiple parallel work streams for an epic

**Pattern**: Read analysis → Spawn agents → Consolidate → Return

**Usage**:
```bash
Task: "Implement epic feature-name with parallel streams"
Agent: parallel-worker
Returns: "Completed 4/4 streams, 15 files modified"
```

**Returns**: Consolidated status of all parallel work

---

## Concurrency Best Practices

### Identify Parallel Work

Look for tasks that:

1. **Modify different files**: Can run in parallel
   - Database schema vs UI components
   - API endpoints vs documentation

2. **Have no dependencies**: No shared state or calls
   - Authentication vs search features
   - Different API endpoints

3. **Can be integrated later**: Independent components
   - Core features before polish
   - Backend before frontend integration

### Avoidism

Don't False Parallel mark as parallel if tasks:

1. **Share configuration files**: `config.yaml`, `.env`
2. **Modify same database schema**: Risk of conflicts
3. **Have shared utilities**: `lib/utils`, `src/helpers`
4. **Depend on each other's output**: Data flow

### Dependency Chains

Model dependency chains clearly:

```
Task 1: Database schema (parallel: true)
  └─ Task 2: API endpoints (parallel: false, depends_on: [1])
      ├─ Task 3: User endpoints (parallel: false, depends_on: [2])
      └─ Task 4: Auth endpoints (parallel: false, depends_on: [2])
          └─ Task 5: Token refresh (parallel: false, depends_on: [4])
```

### Conflict Mitigation

When `conflicts_with` is set:

1. **Plan integration early**: Schedule conflict resolution
2. **Communicate with other agents**: Coordinate file changes
3. **Use merge strategies**: Git merge or rebase as needed
4. **Test integration**: Verify combined changes work

---

## Quick Reference

### Common Commands

| Action | Command |
|--------|---------|
| Create PRD | Write to `prds/name.md`, commit |
| Create epic | `gh issue create --label epic` |
| Create task | `gh issue create --label task` |
| Add task list | Edit epic issue with `- [ ] #NUM` |
| Create worktree | `git worktree add ../epic-name -b epic/name` |
| Remove worktree | `git worktree remove ../epic-name` |
| Commit work | `git commit -m "Task #NUM: Description"` |
| Update issue | `gh issue comment NUM --body "@FILE"` |
| Close issue | `gh issue close NUM` |

### File Locations

| What | Where |
|------|-------|
| PRDs | `prds/` |
| Configuration | `.opencode/opencode.json` |
| Skills | `.opencode/skills/` |
| Agents | `.opencode/agents/` |

### Issue Labels

```
epic           - Epic-level issue
task           - Individual task
feature-name   - Feature scope
status:open    - Not started
status:in-progress - Active work
status:blocked - Blocked
status:done    - Complete
```

---

## Error Handling

### GitHub CLI Errors

```bash
# Authentication failure
gh issue create && echo "❌ GitHub auth failed. Run: gh auth login"

# Repository not found
gh issue create && echo "❌ Check repo. Run: gh repo view"

# Issue not found
gh issue view 123 || echo "❌ Issue #123 not found"
```

### Worktree Errors

```bash
# Worktree already exists
git worktree add ../epic-name 2>&1 | grep "already exists"

# Branch already exists
git checkout -b epic/name 2>&1 | grep "already exists"
```

### Concurrency Errors

If `depends_on` task doesn't exist:
```
❌ Task #123 depends on #456 which doesn't exist
Create #456 first or update depends_on field
```

---

## Getting Started

### New Project

1. Copy this AGENTS.md to project root
2. Create `prds/` directory
3. Configure `.opencode/opencode.json`:
   ```json
   {
     "instructions": ["./AGENTS.md"]
   }
   ```
4. Create first PRD for your initial feature

### Existing Project

1. Copy this AGENTS.md to project root
2. Create `prds/` directory
3. Configure `.opencode/opencode.json`
4. Identify features needing structured planning
5. Create PRD for next feature

---

## Tips

1. **Keep PRDs focused**: One feature per PRD
2. **Break tasks small**: 1-3 day tasks ideal
3. **Mark parallelism**: Enables faster delivery
4. **Link everything**: Commits → Issues → PRDs
5. **Update progress**: Comment on issues as you work
6. **Use worktrees**: Keep main branch clean
7. **Commit often**: Small commits with clear messages

---

**Remember**: Every line of code should trace back to a requirement. OSPM makes that traceable.
