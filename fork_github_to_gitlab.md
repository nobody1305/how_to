
# Forking a GitHub Repository to GitLab

## Step 1: Clone the GitHub Repository Locally

```bash
git clone https://github.com/username/repo-name.git
cd repo-name
```

## Step 2: Create a New Repository on GitLab

1. Go to https://gitlab.com
2. Click **"New project"**
3. Select **"Create blank project"**
4. Set project name, visibility, etc.
5. Click **"Create project"**

## Step 3: Add GitLab as a Remote

```bash
git remote rename origin github
git remote add origin https://gitlab.com/yourname/repo-name.git
```

## Step 4: Push Code to GitLab

```bash
git push -u origin --all      # Push all branches
git push -u origin --tags     # Push all tags
```

### 🔧 Troubleshooting: Push Rejected Error

**Error Message**:
```text
! [rejected] main -> main (fetch first)
error: failed to push some refs ...
hint: Updates were rejected because the remote contains work that you do not have locally.
```

#### Option 1: Force Push (Overwrite GitLab history)

> Use this if you want to fully replace the GitLab repository contents.

```bash
git push -u origin --all --force
git push -u origin --tags --force
```

#### Option 2: Pull First (Merge with GitLab)

> Use this if the GitLab repo has contents you want to keep.

```bash
git pull origin main --allow-unrelated-histories
# Resolve merge conflicts if prompted
git push -u origin --all
git push -u origin --tags
```

---

## Optional: Set GitLab as Default Remote

```bash
git remote remove github
```
