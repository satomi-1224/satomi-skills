# satomi-skills

satomi. の作業規約を Claude Code のスキルとエージェントとして束ねたリポジトリ。

**スキル 1 つに対しエージェント 1 つを対応させる。** スキルが規約の出典を担い、エージェントが手順と制約を担う。

## 収録内容

| スキル | エージェント | 担当範囲 |
|---|---|---|
| `/satomi-skills:core` | `@satomi-skills:core` | 作業全体の基準。規範・開発スタイル・知識の扱い・言語 |
| `/satomi-skills:commit` | `@satomi-skills:commit` | コミットの粒度と規約。変更を type 単位に分割して積む |
| `/satomi-skills:worktree` | `@satomi-skills:worktree` | 元の clone を汚さない worktree の配置と後始末 |
| `/satomi-skills:document-design` | `@satomi-skills:document-design` | 文体・配色・書体・レイアウト・印刷 CSS の規約 |
| `/satomi-skills:skill-builder` | `@satomi-skills:skill-builder` | このリポジトリへのスキルとエージェントの追加手順 |
| `/satomi-skills:skill-refactor` | `@satomi-skills:skill-refactor` | 長くなったスキルの計測と `references/` への分割 |

## 導入

### Claude Code に入れる

このリポジトリはマーケットプレイス兼プラグインである。2 コマンドで入る。

```
/plugin marketplace add satomi-1224/satomi-skills
/plugin install satomi-skills@satomi-skills
```

ターミナルからでも同じ操作ができる。

```
claude plugin marketplace add satomi-1224/satomi-skills
claude plugin install satomi-skills@satomi-skills
```

入ったことを確認する。`Skills` と `Agents` がどちらも 6 と出れば成功である。

```
claude plugin details satomi-skills
```

インストール範囲は `--scope` で選ぶ。既定は `user`。

| scope | 適用範囲 | 設定の書き込み先 |
|---|---|---|
| `user`（既定） | 全プロジェクト | `~/.claude/settings.json` |
| `project` | このプロジェクトのみ。リポジトリで共有できる | `.claude/settings.json` |
| `local` | このプロジェクトのみ。自分だけ | `.claude/settings.local.json` |

### private リポジトリの認証

このリポジトリは private である。導入するアカウントに読み取り権限が必要になる。

`owner/repo` の短縮形は **既定で SSH でクローンする。** SSH 鍵が `ssh-agent` に載っていて、`github.com` が `known_hosts` にある状態が前提である。Claude Code は SSH の対話プロンプトを抑止するため、鍵のパスフレーズ入力待ちになると失敗する。

HTTPS でクローンさせたい場合は環境変数で切り替える。git の credential helper（`gh auth setup-git` や osxkeychain）が使われる。

```
CLAUDE_CODE_PLUGIN_PREFER_HTTPS=1
```

private リポジトリでは背景での自動更新が不安定になる。背景更新は credential helper を無効化して `git pull` するため、HTTPS では認証できずクローンのやり直しにフォールバックする。SSH リモートなら影響しない。HTTPS で運用するなら次を設定して、失敗時に既存のクローンを保持させる。

```
CLAUDE_CODE_PLUGIN_KEEP_MARKETPLACE_ON_FAILURE=1
```

### ローカルのクローンから試す

手を入れながら使う場合は、クローンしたディレクトリを直接マーケットプレイスとして登録する。編集は数秒で反映され、再起動は要らない。

```
git clone https://github.com/satomi-1224/satomi-skills
claude plugin marketplace add ./satomi-skills --scope local
claude plugin install satomi-skills@satomi-skills --scope local
```

`--scope local` にしておくと、そのプロジェクトの `.claude/settings.local.json` にだけ書かれるので他に影響しない。外すときは `claude plugin marketplace remove satomi-skills --scope local`。

### 個別のスキルだけ取り込む

プラグイン全体ではなく一部だけ使う場合は、スキルディレクトリをコピーする。呼称は `/satomi-skills:commit` ではなく `/commit` になる。

```
cp -R satomi-skills/skills/commit ~/.claude/skills/      # 全プロジェクトで使う
cp -R satomi-skills/skills/commit .claude/skills/        # このプロジェクトだけ
```

`references/` などの同梱ファイルごとコピーする。

対応するエージェントも使う場合は `agents/commit.md` を `~/.claude/agents/` に置く。このとき **`skills:` の書き換えが必要になる。** `satomi-skills:` 付きの参照はプラグインのスキルにしか解決しないため、個人スキルとして置いたなら接頭辞を外す。`core` も参照しているので合わせてコピーする。

```yaml
# プラグインとして入れた場合（そのまま）
skills:
  - satomi-skills:core
  - satomi-skills:commit

# 個人スキルとしてコピーした場合（接頭辞を外す）
skills:
  - core
  - commit
```

### Claude Code 以外の Agent Skills 対応クライアント

`SKILL.md` は [Agent Skills 標準](https://agentskills.io/specification)に収めてあるため、`skills/<name>/` をディレクトリごとクライアントのスキルディレクトリへコピーすれば動く。書き換えは不要。

```
git clone https://github.com/satomi-1224/satomi-skills
cp -R satomi-skills/skills/commit <クライアントのスキルディレクトリ>/
```

`agents/` は Claude Code 固有のため、他クライアントでは読み込まれない。手順と規約はスキル側に置く設計なので、スキル単体でも規約の出典として機能する。

### APM (Agent Package Manager)

[APM](https://github.com/microsoft/apm) からも標準の Agent Skills として取得できる。

```
apm install satomi-1224/satomi-skills
apm install satomi-1224/satomi-skills --skill commit
```

## 使い方

| したいこと | 書き方 | 挙動 |
|---|---|---|
| スキルを明示的に呼ぶ | `/satomi-skills:commit` | 規約が現在の文脈に読み込まれる |
| エージェントに委譲する | `@satomi-skills:commit` | 別コンテキストで作業し、結果を返す |
| 自動で任せる | 何も書かない | `description` に合致したときに Claude が自ら読み込む |

スキルとエージェントは 1 対 1 で対応している。規約を自分で見ながら作業したいときはスキル、丸ごと任せたいときはエージェントを使う。

## 管理

| 操作 | コマンド |
|---|---|
| 一覧 | `claude plugin list` |
| 内訳とトークンコスト | `claude plugin details satomi-skills` |
| 更新 | `/plugin marketplace update satomi-skills` |
| 一時的に無効化 | `claude plugin disable satomi-skills` |
| 再有効化 | `claude plugin enable satomi-skills` |
| 削除 | `claude plugin uninstall satomi-skills` |
| マーケットプレイスごと削除 | `claude plugin marketplace remove satomi-skills` |

プラグインの更新は再起動で適用される。

## 構成

```
satomi-skills/
├── .claude-plugin/
│   ├── plugin.json          # プラグイン定義（name: satomi-skills）
│   └── marketplace.json     # 配布定義（source: "./" で自己参照）
├── skills/<name>/SKILL.md   # スキル本体。同梱ファイルは同ディレクトリに置く
├── agents/<name>.md         # スキルに対応するエージェント
└── README.md
```

## 設計方針

| 方針 | 理由 |
|---|---|
| `SKILL.md` の frontmatter は Agent Skills 標準の 6 フィールドのみ | 標準外フィールドを入れると claude.ai / Skills API へのパッケージングがハードエラーで失敗する。6 フィールドに収めれば他クライアントからもそのまま読める |
| Claude Code 固有の設定はエージェント側に寄せる | `model` `effort` `tools` はエージェントの frontmatter で指定できる。スキルの移植性を落とさずに同じ制御ができる |
| エージェントは `agents/` 直下に置く | `skills/<name>/` 配下に置いて `plugin.json` の `agents` に列挙すると、マニフェスト検証は通るが実行時にロードされない |
| `disable-model-invocation` を使わない | 標準外フィールドである上に、この指定が付いたスキルはエージェントにプリロードできない |
| 対応関係を双方向に記録する | スキルとエージェントが別ディレクトリに分かれるため。スキル側は `metadata.paired-agent`、エージェント側は `skills:` に相手を書く |
| `core` を全エージェントにプリロードする | スキル本文は呼び出されたときに読み込まれる。常に効かせたい基準は `skills:` へのプリロードで担保する |
| スキルが「何をどうするか」、エージェントが「誰であるか」 | 手順と規約はスキルにだけ置き、エージェント本文はペルソナ 1 段落に限る。対応付けは `skills:` が宣言するので散文で書き直さない |
| `SKILL.md` の on-invoke を 5,000 トークン以内に保つ | Agent Skills 仕様の推奨値。条件付きでしか使わない実装詳細は `references/` に出し、読まれたときだけコストを払う |

対応関係は grep で辿れる。

```
grep -r "paired-agent" skills/
grep -rA2 "^skills:" agents/
```

## 個人設定

作業の基準は `core` スキルが持つため、`CLAUDE.md` は不要である。

環境側の設定はプラグインでは配布できない（プラグインの `settings.json` は `agent` と `subagentStatusLine` しか受け付けない）。`~/.claude/settings.json` に直接置く。以下は作者の設定値である。

```json
{
  "language": "japanese",
  "alwaysThinkingEnabled": false,
  "effortLevel": "xhigh",
  "skipDangerousModePermissionPrompt": true,
  "skipAutoPermissionPrompt": true,
  "permissions": {
    "defaultMode": "auto"
  }
}
```

## 検証

```
claude plugin validate .claude-plugin/plugin.json --strict
claude plugin validate .claude-plugin/marketplace.json --strict
claude plugin details satomi-skills
```

`claude plugin details` の `Skills` と `Agents` の数が `skills/` と `agents/` の実数に一致すること。マニフェスト検証だけでは配置の誤りを検出できない。

## 参考文献

- [Agent Skills 仕様](https://agentskills.io/specification)
- [Claude Code — Skills](https://code.claude.com/docs/en/skills)
- [Claude Code — Subagents](https://code.claude.com/docs/en/sub-agents)
- [Claude Code — Plugins reference](https://code.claude.com/docs/en/plugins-reference)
- [Claude Code — Plugin marketplaces](https://code.claude.com/docs/en/plugin-marketplaces)
- [APM (Agent Package Manager)](https://github.com/microsoft/apm)
