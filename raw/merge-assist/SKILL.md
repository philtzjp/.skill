---
name: merge-assist
description: 対話型マージコンフリクト解決ツール。コンフリクトマーカーを読み取り、各コンフリクトをコンテキスト付きで提示し、ユーザーの選択に従い自動解決する。/merge-assistで起動。
---

# Merge Assist

The keywords "MUST", "NEVER", "SHOULD", and "MAY" in this document are to be interpreted as described in RFC 2119.

## Overview

対話形式でマージコンフリクトを解決するスキル。各コンフリクトの両サイドを提示し、ユーザーに選択を求めて自動解決する。

## Procedure

### 1. コンフリクトファイルの検出

MUST run the following to find all conflicted files:

```bash
git diff --name-only --diff-filter=U
```

If no conflicts are found, report "コンフリクトはありません" and exit.

### 2. ブランチ名の取得

MUST determine the actual branch names for clear labeling:

```bash
# Current branch name
git rev-parse --abbrev-ref HEAD

# Merge source — extract from conflict marker or MERGE_MSG
head -1 .git/MERGE_MSG 2>/dev/null || echo "unknown"
```

Use these real branch names (e.g. `main`, `feature/auth`) throughout the UI instead of Git jargon like "ours"/"theirs".

### 3. コンフリクトの解析

Each conflicted file MUST be read in full. Parse all conflict blocks delimited by:

```
<<<<<<< HEAD (or branch name)
... current branch side ...
=======
... incoming branch side ...
>>>>>>> branch-name
```

For each file, extract:
- The file path
- Each conflict block with surrounding context (3 lines before/after)
- The current branch side content
- The incoming branch side content

### 4. 対話的な解決

For each conflict block, MUST first summarize what each side does in plain Japanese. Then present the following to the user, using the **real branch names** obtained in step 2:

```
## [ファイル名] コンフリクト N/M

`fetchUser()` を使った認証処理と、`getUser()` を使った新しい認証処理が競合しています。

### `main` ブランチ（現在）:
<current branch code>

### `feature/auth` ブランチ（取り込み）:
<incoming branch code>

どちらを残しますか？
1. `main` の内容を残す
2. `feature/auth` の内容を残す
3. 両方残す（`main` → `feature/auth` の順）
4. 両方残す（`feature/auth` → `main` の順）
5. 手動で編集する
```

MUST use AskUserQuestion to get the user's choice for each conflict.

When the user responds:
- **Option 1**: Keep only the current branch side
- **Option 2**: Keep only the incoming branch side
- **Option 3**: Keep both, current first then incoming
- **Option 4**: Keep both, incoming first then current
- **Option 5**: Ask the user to provide the replacement text, then apply it

### 4. コンフリクトの解決

After the user chooses for each conflict block:

1. MUST use the Edit tool to replace the entire conflict block (including markers) with the chosen content
2. MUST preserve the original indentation and whitespace
3. After resolving all conflicts in a file, MUST run `git diff <file>` to show the result
4. After all files are resolved, MUST run `git add` on each resolved file

### 5. 完了報告

After all conflicts are resolved, MUST report:

```
## マージコンフリクト解決完了

- 解決ファイル数: N
- 解決コンフリクト数: M
- ステージ済み: Yes

次のステップ: `git commit` でマージを完了してください
```

### 6. Prohibitions

- NEVER auto-resolve conflicts without asking the user
- NEVER discard changes silently
- NEVER run `git commit` automatically — the user decides when to commit
- NEVER modify lines outside of conflict markers
