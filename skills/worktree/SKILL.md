---
name: worktree
description: worktree を ghq root 直下の .worktrees/<host>/<user>/<repo>/<branch> に集約する配置規則と、作業後も worktree を削除しない後始末の方針を定める。
license: MIT
allowed-tools: Bash(ghq root) Bash(git worktree *) Bash(git branch *) Bash(git config *) Bash(git rev-parse *)
metadata:
  paired-agent: satomi-skills:worktree
---

# worktree 作業手順
元の clone を汚さず worktree で作業するための手順を示す

## 方針
- 元の clone は人間が確認するために存在する。Claude は元の clone のブランチ・作業ツリーを直接変更しない。
- 作業タスクに着手するときは worktree を作成し、その中で作業する。

## 配置先
- worktree は ghq root 直下の `.worktrees/` に集約する。
- パス: `<ghq-root>/.worktrees/<host>/<user>/<repo>/<branch>`
  - 例: `~/ghq/.worktrees/github.com/satomi-1224/satomi-skills/feature-x`
- 元の clone の隣（ghq の `<host>/<user>/` 階層）には作らない。`ghq list` に混ざり、リポジトリ選択を汚すため。
- `.worktrees`（ドット始まり）は ghq の列挙対象から外れる。

## 命名
- ブランチ名をそのままディレクトリ名に使う。
- ブランチ名に `/` が含まれる場合（例 `feature/foo`）はディレクトリ階層になるため、そのまま使ってよい。

## 作成手順
1. ghq root を取得する。
   ```
   ghq root
   ```
   `git config --get ghq.root` は `ghq.root` を明示設定していない環境では何も返さない。
   既定値（`~/ghq`）で運用している場合も `ghq root` は実効値を返すため、こちらを使う。
2. 現在のリポジトリの `<host>/<user>/<repo>` を求める（元 clone のパスから ghq root を除いた部分）。
3. worktree のパスを組み立て、新規ブランチを切って作成する。
   ```
   git worktree add <ghq-root>/.worktrees/<host>/<user>/<repo>/<branch> -b <branch>
   ```
4. 以降の作業はこの worktree 内で行う。

## 後始末
- 作業完了後も worktree は残す。人間がマージ・確認するために存在するため。
- Claude は `git worktree remove` で削除しない。
- 明示的に削除を依頼・許可している場合は削除を行う。
