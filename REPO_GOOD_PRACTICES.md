# Repo Good Practices

## Daily Workflow

```bash
# 1. Update main
git checkout main && git pull

# 2. Create branch
git checkout -b <branch-name>

# 3. Work and commit
git add .
git commit -m "message"

# 4. Push branch
git push -u origin <branch-name>

# 5. Create PR on GitHub, merge, then cleanup locally:
git checkout main
git pull
git branch -d <branch-name>
```

## Useful Commands

```bash
# Check current branch
git branch

# List all branches (local + remote)
git branch -a

# Check status
git status

# View commit history
git log --oneline -10

# See what changed
git diff

# Discard local changes
git checkout -- <file>

# Undo last commit (keep changes)
git reset --soft HEAD~1
```

## After Merging PR

```bash
git checkout main
git pull
git branch -d <merged-branch>      # delete local
git remote prune origin            # cleanup stale remote refs
```

## Branch Naming

```plain-text
feature/add-etcd-notes
fix/typo-in-readme
docs/update-scheduling-section
```

## Commit Best Practices

### Format

```plain-text
<type>: <short description>

[optional body]
```

### Types

| Type | Use For |
|------|---------|
| `feat` | New feature or content |
| `fix` | Bug fix or correction |
| `docs` | Documentation only |
| `refactor` | Restructure without changing content |
| `style` | Formatting, typos |
| `chore` | Maintenance tasks |

### Good Examples

```bash
git commit -m "feat: add ETCD backup/restore notes"
git commit -m "fix: correct API server port number"
git commit -m "docs: add mermaid diagram for scheduling flow"
git commit -m "style: fix markdown formatting in control-plane.md"
```

### Bad Examples

```bash
git commit -m "update"           # too vague
git commit -m "fixed stuff"      # not descriptive
git commit -m "WIP"              # don't commit incomplete work
git commit -m "asdfgh"           # just no
```

### Tips

- Keep subject line under 50 characters
- Use imperative mood: "add" not "added"
- One logical change per commit
- Commit often, push when ready
