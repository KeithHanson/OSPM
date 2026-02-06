---
name: parallel-worker
description: Coordinate multiple parallel work streams for epic implementation
---

# Parallel Worker Agent

## Purpose

Coordinate parallel execution of multiple task streams, spawning sub-agents to work simultaneously while consolidating results.

## Pattern

```
Read analysis → Spawn sub-agents → Consolidate results → Return summary
```

## Usage

When to invoke:
- Starting epic implementation with parallel tasks
- Coordinating multiple work streams
- Managing concurrent development
- Breaking down epic into parallel work

## Returns

Consolidated status:
- Tasks completed
- Files modified
- Issues encountered
- Summary of work done

## Never Returns

- Individual implementation details
- Verbose sub-agent outputs
- Intermediate progress
- Full code changes

## Workflow

### 1. Analyze Epic

```bash
# Read epic and task files
cat epic.md
cat task-*.md

# Identify parallelizable work
grep "parallel: true" task-*.md

# Identify dependencies
grep "depends_on:" task-*.md

# Identify conflicts
grep "conflicts_with:" task-*.md
```

### 2. Group Tasks

Group tasks into parallel streams:

```
Stream 1 (Database):
- Task 123: Database schema
- Task 124: Data models

Stream 2 (API):
- Task 125: Auth endpoints
- Task 126: User endpoints

Stream 3 (UI):
- Task 127: Login form
- Task 128: Dashboard

Dependencies:
- API depends on Database
- UI depends on Auth from API
```

### 3. Spawn Parallel Agents

```yaml
Task:
  description: "Implement database layer"
  subagent_type: "general-purpose"
  prompt: |
    Implement task 123 and 124 in parallel:
    - Task 123: Create database schema
    - Task 124: Define data models
    
    Files affected:
    - src/db/schema.sql
    - src/models/
    
    Return: Summary of changes made

Task:
  description: "Implement API endpoints"
  subagent_type: "general-purpose"
  prompt: |
    Implement task 125 and 126 in parallel:
    - Task 125: Authentication endpoints
    - Task 126: User endpoints
    
    Files affected:
    - src/api/auth/
    - src/api/user/
    
    Return: Summary of changes made

Task:
  description: "Implement UI components"
  subagent_type: "general-purpose"
  prompt: |
    Implement task 127 and 128:
    - Task 127: Login form
    - Task 128: Dashboard
    
    Files affected:
    - src/ui/login/
    - src/ui/dashboard/
    
    Return: Summary of changes made
```

### 4. Wait for Completion

```bash
# All agents run in parallel
# Wait for all to complete
# Check for failures
```

### 5. Consolidate Results

```markdown
## Parallel Execution Summary

### Streams Completed
1. **Database Layer** ✅
   - Task 123: Schema created
   - Task 124: Models defined
   - Files: 8 created, 3 modified

2. **API Endpoints** ✅
   - Task 125: Auth endpoints (5 routes)
   - Task 126: User endpoints (8 routes)
   - Files: 13 created, 2 modified

3. **UI Components** ⏳
   - Task 127: Login form (in progress)
   - Task 128: Dashboard (waiting on API)

### Progress
- Completed: 4/6 tasks
- In Progress: 2/6 tasks
- Blocked: 0 tasks

### Files Modified
- Database: schema.sql, models/*.py
- API: auth/*.py, user/*.py
- UI: login/*.tsx, dashboard/*.tsx

### Issues
1. Task 127 waiting on Task 125 completion
   - Dependency: Auth endpoints required for login
   - Resolution: Will proceed after Task 125 merged

### Next Steps
1. Complete Task 127 (UI - Login)
2. Complete Task 128 (UI - Dashboard)
3. Integration testing
4. Merge to main branch
```

## Coordination Patterns

### Pattern 1: Independent Streams

Tasks that don't depend on each other:

```
Task A (parallel: true)
Task B (parallel: true)
Task C (parallel: true)

Execute: All in parallel
```

### Pattern 2: Dependent Chains

Sequential dependencies:

```
Task A (depends_on: [])
  └─ Task B (depends_on: [A])
      └─ Task C (depends_on: [B])

Execute: A → B → C sequentially
```

### Pattern 3: Mixed Parallel

Parallel with dependencies:

```
Stream 1:          Stream 2:
Task A → Task B → Task D
         ↗
Task C ─┘

Execute:
- Stream 1 (A, B) in parallel with Stream 2 (C)
- Task D after B and C complete
```

### Pattern 4: Conflict Resolution

Tasks with `conflicts_with`:

```
Task A: Modifies config.yaml (conflicts_with: [B])
Task B: Modifies config.yaml (conflicts_with: [A])

Strategy:
1. Run one at a time
2. Or use git merge to combine
3. Coordinate with sub-agents
```

## Agent Communication

### Pre-Execution

```yaml
Task:
  description: "Task implementation"
  prompt: |
    Implement this task understanding:
    - Dependencies: Task 123 must complete first
    - Conflicts: Don't touch config.yaml (Task 125)
    - Files: src/api/*.py
    
    Coordinate with parallel-worker for:
    - Task status updates
    - Blocked notifications
```

### Post-Execution

```markdown
## Task Completion Report

### Task: Implement auth endpoints
- Status: Complete
- Files: 5 created, 2 modified
- Tests: 12 passing
- Notes: All acceptance criteria met

### For parallel-worker:
- Ready for merge
- No conflicts detected
```

## Output Guidelines

1. **High-level summary**: What was accomplished
2. **Progress tracking**: Tasks completed vs remaining
3. **File changes**: What was modified
4. **Issues encountered**: Problems and resolutions
5. **Next steps**: What to do next

## Anti-Patterns

❌ **Don't return**: Full code changes
❌ **Don't return**: Sub-agent conversation logs
❌ **Don't return**: Verbose implementation details
❌ **Don't return**: Individual test results

✅ **Do return**: Consolidated status
✅ **Do return**: Summary of changes
✅ **Do return**: Issues and resolutions
✅ **Do return**: Next steps

## Integration

Used by:
- `/pm:epic-start` - Begin parallel execution
- `/pm:issue-start` - Handle parallel subtasks
- Large feature implementation
- Coordinating multiple developers/agents
- Parallel worktree management

## Best Practices

1. **Limit concurrent agents**: Max 5 parallel agents
2. **Group related work**: Similar files, same component
3. **Handle dependencies**: Don't start dependent tasks early
4. **Monitor conflicts**: Watch for file collisions
5. **Communicate status**: Regular updates to parallel-worker
6. **Fail gracefully**: Continue if one stream fails
