---
name: semantic-commit
description: gitコミット時にセマンティックコミット規約を強制する。コミット、変更保存、git add、ステージング時に自動発動。
---

# Semantic Commit

The keywords "MUST", "NEVER", "SHOULD", and "MAY" in this document are to be interpreted as described in RFC 2119.

## Message Format

```
type: short description in Japanese
```

For monorepos (multiple packages):

```
type(scope): short description in Japanese
```

### Prefixes

| Prefix | Usage |
|---|---|
| `feat` | New feature |
| `fix` | Bug fix |
| `perf` | Performance improvement |
| `refactor` | Code improvement without functional changes |
| `docs` | Documentation changes |
| `style` | Code style fixes |
| `test` | Adding or modifying tests |
| `chore` | Miscellaneous tasks |
| `ci` | CI/CD configuration changes |
| `build` | Build configuration changes |

## Commit Procedure

### 1. Analyze Changes

Run `git status` and `git diff` to review changes, then group them by scope.

You MUST split commits by logical scope (package, feature). You MUST NOT bulk-commit unrelated changes together, even if 20 files have changed.

### 2. Determine Author

Analyze `git diff` for each file to determine who wrote the changes:

- **Only Claude-generated changes** — MUST set both author and committer to Claude:
  ```
  GIT_COMMITTER_NAME="Claude" GIT_COMMITTER_EMAIL="noreply@anthropic.com" \
  git commit --author="Claude <noreply@anthropic.com>"
  ```
- **Contains user-edited changes** — MUST NOT override author or committer (uses the user's git config)
- When uncertain, SHOULD ask the user

### 3. Stage and Commit

1. `git add` target files (per scope)
2. `git commit`

### 4. Prohibitions

- NEVER add `Co-Authored-By`
- NEVER use `git add .` or `git add -A`
- NEVER mix unrelated changes in a single commit

## Example

```bash
# Scope 1: API fix
git add packages/api/src/auth.ts packages/api/src/middleware.ts
GIT_COMMITTER_NAME="Claude" GIT_COMMITTER_EMAIL="noreply@anthropic.com" \
git commit --author="Claude <noreply@anthropic.com>" -m "fix(api): 認証ミドルウェアのトークン検証を修正"

# Scope 2: Frontend feature
git add packages/web/src/pages/settings.vue packages/web/src/components/SettingsForm.vue
GIT_COMMITTER_NAME="Claude" GIT_COMMITTER_EMAIL="noreply@anthropic.com" \
git commit --author="Claude <noreply@anthropic.com>" -m "feat(web): 設定ページを追加"
```
