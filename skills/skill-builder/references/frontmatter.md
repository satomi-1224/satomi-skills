# frontmatter リファレンス

スキルとエージェントの frontmatter の全仕様。最小書式は `SKILL.md` にある。

## スキル — Agent Skills 標準の 6 フィールド

`SKILL.md` に書けるのはこの 6 つだけとする。

| フィールド | 必須 | 制約 |
|---|---|---|
| `name` | ✓ | 64 文字以内。小文字英数とハイフンのみ。先頭・末尾のハイフンと連続ハイフン（`--`）は不可。**親ディレクトリ名と一致させる** |
| `description` | ✓ | 1024 文字以内。何をするか + いつ使うかを書く。自動起動の判断に使われる |
| `allowed-tools` | | 承認なしで許可するツール。空白区切りの文字列 |
| `metadata` | | 文字列から文字列への写像。このリポジトリでは `paired-agent` を入れる |
| `license` | | ライセンス名か同梱ファイル名 |
| `compatibility` | | 500 文字以内。特別な実行環境要件があるときだけ書く。大半のスキルには不要 |

### `description` の書き方

自動起動の判断キーになる。**何をするか + いつ使うか + 想起されるキーワード**を入れる。

```yaml
# 良い
description: コミットを作成する、コミットメッセージを起案する、コミットの粒度や分割を判断するときに使う。type は feat/fix/docs/... に限り、1 コミット = 1 type に収める分割規則を定める。

# 悪い
description: コミットを手伝う
```

## 6 フィールドに絞る理由

Claude Code は標準外のフィールドも受け付ける。だが**標準外を入れると claude.ai / Skills API / `package_skill.py` へのパッケージングがハードエラーで失敗する。**

```
Unexpected key(s) in SKILL.md frontmatter: argument-hint.
Allowed properties are: allowed-tools, compatibility, description, license, metadata, name
```

6 フィールドに収めておけば、Claude Code 以外の Agent Skills 対応クライアントからもそのまま読める。

### 使わない Claude Code 固有フィールドと代替

| 使わないもの | 代替 |
|---|---|
| `model` / `effort` | エージェントの frontmatter に書く |
| `disable-model-invocation` | 使わない。付けるとエージェントにプリロードできなくなる |
| `when_to_use` | `description` に畳む |
| `argument-hint` / `arguments` | 本文に書く |
| `paths` | `description` に適用条件を書く |
| `context: fork` / `agent` | 対応するエージェントに委譲する構成で代替する |
| `hooks` | プラグインの `hooks/hooks.json` に置く |

## `allowed-tools` の記法

Claude Code の権限ルール構文と同じ。

| 書き方 | 意味 |
|---|---|
| `Bash(git status *)` | `git status ` で始まるコマンド。末尾の ` *` は語境界を強制する |
| `Bash(git status:*)` | 上と等価。`:*` は**パターン末尾でのみ**有効 |
| `Bash(git:* push)` | コロンが literal 扱いになり意図どおり動かない |
| `Bash(npm run build)` | 完全一致 |
| `Bash(ls *)` | `ls -la` にマッチし `lsof` にはマッチしない（語境界あり） |
| `Bash(ls*)` | `ls -la` と `lsof` の両方にマッチする（語境界なし） |

注意点。

- **引数を絞る条件は書かない。** `&&` `||` `;` `|` `&` と改行で分割されたコマンドは各サブコマンドが個別に照合される。細かい制約は破られやすく壊れやすい。
- `timeout` `time` `nice` `nohup` `stdbuf` `command` `builtin` `noglob` と引数なしの `xargs` は照合前に除去される。`Bash(npm test *)` は `timeout 30 npm test` にもマッチする。
- リダイレクト先（`>` `>>` `2>`）はファイル書き込みとして別に検査される。コマンドを許可しても書き込み先は許可されない。

## エージェント

Claude Code 固有。標準に縛られないので必要なだけ使う。

| フィールド | 用途 |
|---|---|
| `name` | 識別子（必須）。コロンは使えない。ファイル名と揃える |
| `description` | いつ委譲するか（必須）。動詞で始め、対象領域を名指しする |
| `skills` | プリロードするスキル。`satomi-skills:<name>` の形で書く |
| `tools` | 許可ツールの allowlist。省略すると全ツールを継承する |
| `disallowedTools` | 禁止ツール。`tools` より先に適用される |
| `model` | `inherit`（既定）/ エイリアス / 完全な ID |
| `effort` | `low` `medium` `high` `xhigh` `max` |
| `color` | `red` `blue` `green` `yellow` `purple` `orange` `pink` `cyan` |
| `maxTurns` | N ターンで停止する |
| `memory` | `user` / `project` / `local`。セッションを越えて記憶する |
| `background` | `true` で常にバックグラウンド実行 |
| `isolation` | `worktree` で一時 worktree 内で実行する |

プラグインが配布するエージェントでは `hooks` `mcpServers` `permissionMode` は無視される。

### `tools` の絞り方

副作用を持たせたくないエージェントでは allowlist で担保する。

| エージェント | `tools` | 理由 |
|---|---|---|
| `commit` | `Bash, Read, Grep, Glob` | `Write` / `Edit` を外し実装変更を構造的に不可能にする |
| `worktree` | `Bash, Read, Grep, Glob` | 同上 |
| `document-design` | 省略（全継承） | 資料生成に広いツールが必要 |
| `skill-builder` | 省略（全継承） | ファイル追加と検証コマンドが必要 |

`Skill` を `tools` から外すと、そのエージェントは他のスキルを一切呼べなくなる。`skills:` のプリロードは `Skill` ツールを必要としないため、絞っても対応スキルは効く。
