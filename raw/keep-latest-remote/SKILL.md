---
name: keep-latest-remote
description: ローカルブランチをリモートと最新に保つ。git操作、ブランチ作業、push、pull、merge、rebase、checkout時に自動発動。
---

# Keep Latest Remote

The keywords "MUST", "NEVER", "SHOULD", and "MAY" in this document are to be interpreted as described in RFC 2119.

## When to Run

You MUST run these checks before starting any git-related work (commit, push, branch creation, merge, rebase).

## Procedure

### 1. Fetch Remote

MUST run `git fetch --prune` before any git operation to ensure local refs are current.

### 2. Validate Current Branch

Check the current branch against remote state:

- **Merged branch detection**: MUST check if the current branch has already been merged into the default branch (main/master). If merged, warn the user and suggest switching.
  ```bash
  git branch --merged origin/main | grep -v 'main\|master'
  ```

- **Deleted remote branch**: MUST check if the remote tracking branch still exists. If deleted, warn the user.
  ```bash
  git branch -vv | grep ': gone]'
  ```

- **Behind remote**: MUST check if the local branch is behind its remote counterpart. If behind, suggest pulling.
  ```bash
  git rev-list --count HEAD..@{upstream}
  ```

### 3. Actions

| Situation | Action |
|---|---|
| Current branch merged into default | MUST warn user, suggest `git checkout main && git pull` |
| Remote tracking branch deleted | MUST warn user, suggest switching to default branch |
| Local branch behind remote | SHOULD suggest `git pull --rebase` |
| Local branch diverged from remote | SHOULD warn user, suggest rebase or merge |
| Everything up-to-date | No action needed |

### 4. Prohibitions

- NEVER silently switch branches without user confirmation
- NEVER run `git pull` or `git rebase` without user approval
- NEVER delete local branches without user confirmation
