
### 7.5 Workflow-Specific Git Operation Recommendations

#### For Trunk-Based Development

```bash
# Daily workflow
git checkout main
git pull --rebase origin main  # Keep linear history

git checkout -b feature/quick-fix
# Make small changes
git commit -m "Add feature X"

# Update before merging
git checkout main
git pull --rebase origin main
git merge feature/quick-fix --ff-only  # Fast-forward only

# Or squash small commits
git merge feature/quick-fix --squash
```

**Key Operations:**
- **Primary:** `git merge --ff-only` or `git merge --squash`
- **Avoid:** Long-lived branches, cherry-picking
- **Rebase:** Use `git pull --rebase` to keep linear history

---

#### For GitHub Flow

```bash
# Feature development
git checkout main
git pull origin main
git checkout -b feature/user-profile

# Multiple commits during development
git commit -m "Add profile UI"
git commit -m "Add profile API"
git commit -m "Add tests"

# Before creating PR, clean up commits
git rebase -i main  # Squash related commits

# Update branch with latest main
git rebase main

# Push (force with lease if rebased)
git push --force-with-lease origin feature/user-profile

# After PR approval, merge to main (GitHub does this)
# GitHub can do: merge commit, squash, or rebase
```

**Key Operations:**
- **Primary:** `git merge` (PR merges)
- **For cleanup:** `git rebase -i` before PR
- **Avoid:** Cherry-picking (except emergencies)

---

#### For GitLab Flow

```bash
# Feature to main
git checkout main
git pull origin main
git checkout -b feature/dashboard

# Development
git add .
git commit -m "Add dashboard component"

# Merge feature to main
git checkout main
git merge feature/dashboard --no-ff

# Main auto-deploys to staging

# After QA approval, merge main to production
git checkout production
git pull origin production
git merge main --no-ff  # Preserve history

# For hotfixes (upstream first!)
git checkout main
git checkout -b hotfix/critical-bug
git commit -m "Fix critical bug"

git checkout main
git merge hotfix/critical-bug --no-ff

# Then cherry-pick to production if urgent
git checkout production
git cherry-pick <commit-hash>  # Only if can't wait for merge
```

**Key Operations:**
- **Primary:** `git merge --no-ff` (preserve history)
- **For hotfixes:** Merge to main first, then cherry-pick to production
- **Avoid:** Rebasing main or production branches

---

### 7.6 Best Practices Summary for Your Organization

Based on your **GitHub Flow + GitLab Flow Hybrid** recommendation:

#### Daily Development Workflow

```bash
# 1. Start new feature
git checkout main
git pull origin main
git checkout -b ST-XXX/FEAT/feature-name

# 2. Develop with small, focused commits
git add specific-file.js
git commit -m "feat: Add user validation logic"

# 3. Before creating PR, update with latest main
git fetch origin
git rebase origin/main  # Rebase to get latest changes

# 4. Clean up commits before PR (optional but recommended)
git rebase -i origin/main  # Squash "WIP" commits

# 5. Push to remote
git push origin ST-XXX/FEAT/feature-name
# Or if you rebased:
git push --force-with-lease origin ST-XXX/FEAT/feature-name

# 6. Create Pull Request on GitHub
# Automated tests run

# 7. After approval, merge to main (use GitHub UI)
# Choose: "Squash and merge" for clean history

# 8. Main auto-deploys to staging

# 9. After QA approval, merge main to production
git checkout production
git pull origin production
git checkout main
git pull origin main
git checkout production
git merge main --no-ff -m "Production release: Deploy features X, Y, Z"
git push origin production
```

#### Hotfix Workflow (< 1 hour to production)

```bash
# 1. Create hotfix from main (not production!)
git checkout main
git pull origin main
git checkout -b ST-XXX/FIX/critical-bug

# 2. Fix the bug
git add .
git commit -m "fix: Resolve payment processing timeout"

# 3. Create PR to main
git push origin ST-XXX/FIX/critical-bug

# 4. Fast-track review and merge to main
# Main deploys to staging

# 5. Quick QA verification in staging

# 6. Merge main to production
git checkout production
git pull origin production
git merge main --no-ff -m "Hotfix: Payment processing fix"
git push origin production

# Total time: < 1 hour
```

#### What Git Operations to Use When

| Situation | Git Operation | Command | Why |
|-----------|--------------|---------|-----|
| **Creating feature branch** | Branch | `git checkout -b ST-XXX/FEAT/name` | Start isolated work |
| **Regular commits during development** | Commit | `git commit -m "message"` | Save progress |
| **Update feature with latest main** | Rebase | `git rebase main` | Keep branch current, linear history |
| **Clean up commits before PR** | Interactive Rebase | `git rebase -i main` | Present clean history to reviewers |
| **Merge feature to main** | Merge (Squash) | `git merge --squash` or GitHub "Squash and merge" | Clean history, single commit per feature |
| **Merge main to production** | Merge (No-FF) | `git merge main --no-ff` | Preserve deployment history |
| **Emergency hotfix** | Merge | Same as feature, just faster | Maintain consistent process |
| **Only if absolutely necessary** | Cherry-Pick | `git cherry-pick <hash>` | Last resort for selective deployment |

#### Anti-Patterns to Avoid

❌ **DON'T** create DEPLOYMENT-date branches anymore
❌ **DON'T** use cherry-pick for regular deployment (error-prone)
❌ **DON'T** rebase main or production branches
❌ **DON'T** merge production back to main
❌ **DON'T** use `git push --force` (use `--force-with-lease`)
❌ **DON'T** have long-lived feature branches (max 2-3 days)

✅ **DO** merge main forward to production (one direction only)
✅ **DO** use rebase for updating feature branches
✅ **DO** use merge for integrating to main/production
✅ **DO** keep commits small and focused
✅ **DO** write clear commit messages
