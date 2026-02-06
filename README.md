# OSPM: OpenCode Simple PM

A lightweight, GitHub-native project management system for AI-assisted development.

## Quick Start

1. Copy these files to your project:
   - `AGENTS.md` → Project root
   - `.opencode/` → Project root

2. Configure `.opencode/opencode.json`:
   ```json
   {
     "instructions": ["./AGENTS.md"]
   }
   ```

3. Create `prds/` directory for Product Requirement Documents

## Installation

### One-liner Install (Recommended)

```bash
cd /path/to/your/project
gh repo clone KeithHanson/OSPM ospm-temp -- -q && rm -rf ospm-temp/.git && mv ospm-temp/.opencode . && rm -rf ospm-temp
```

This clones the repo, removes the `.git` folder, moves `.opencode` to your project, and cleans up.

### Requirements

- [GitHub CLI](https://cli.github.com/) - `gh` command
- Git authentication configured (`gh auth login`)

## Directory Structure

```
your-project/
├── README.md              # Your project documentation
├── AGENTS.md              # ← OSPM: All-in-one project context
├── prds/                  # ← PRD storage (local)
└── .opencode/
    ├── opencode.json     # Configuration
    ├── skills/
    │   └── github-ops/   # GitHub CLI operations
    └── agents/
        ├── code-analyzer/
        ├── file-analyzer/
        ├── test-runner/
        └── parallel-worker/
```

## Documentation

- **[AGENTS.md](./AGENTS.md)**: Complete OSPM documentation including:
  - Workflow guide
  - PRD, Epic, and Task templates
  - GitHub operations patterns
  - Worktree conventions
  - Specialized agents

## Features

- **GitHub-native**: Issues track work, commits document changes
- **Minimal local state**: Only PRDs stored locally
- **Concurrency analysis**: Identify parallelizable work
- **Full traceability**: From idea to shipped code
- **All-in-one**: Single AGENTS.md file for everything

## Commands Reference

### Create PRD
```bash
# 1. Write PRD to prds/feature-name.md
# 2. Commit
git add prds/feature-name.md
git commit -m "PRD: Add feature-name"
```

### Create Epic Issue
```bash
# Create GitHub issue with epic label
gh issue create --title "Epic: feature-name" --label epic
```

### Create Task Issues
```bash
# Create task with task label
gh issue create --title "Task: task-title" --label task
```

### Create Worktree
```bash
git worktree add ../epic-feature -b epic/feature
```

## Workflow

```
PRD (local) → Epic (GitHub) → Tasks (GitHub) → Worktree → Commit → Ship
```

See [AGENTS.md](./AGENTS.md) for detailed workflow instructions.
