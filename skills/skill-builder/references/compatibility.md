# 他クライアントと APM の互換性

**構成を変えたくなったら、変更前にこれを読む。** 過去に検討して却下した選択肢の記録である。

## 現構成が満たしているもの

| 対象 | 満たし方 |
|---|---|
| Claude Code | `.claude-plugin/` + ルート直下の `skills/` `agents/`。マーケットプレイスは `source: "./"` で自己参照 |
| Agent Skills 標準 | `SKILL.md` の frontmatter を 6 フィールドに限定 |
| APM (Agent Package Manager) | 素の Agent Skills リポジトリとしてそのまま install できる |
| claude.ai / Skills API | 標準 6 フィールドのためパッケージング可能 |

同じ形をとっている先例。

- `anthropics/skills` — `.claude-plugin/` + ルート `skills/`。`apm.yml` なし
- `vercel-labs/agent-skills` — ルート `skills/`。`apm.yml` なし。APM が drop-in 対象として明記している

## 却下: `.apm/` レイアウトへの移行

APM の公開パッケージの正典レイアウトは以下である（`microsoft/apm-sample-package` を実査）。

```
package/
├── apm.yml
└── .apm/
    ├── skills/<name>/SKILL.md
    ├── agents/<name>.agent.md
    ├── instructions/<name>.instructions.md
    └── prompts/<name>.prompt.md
```

採用しない理由。

1. **Claude Code の探索パスと衝突する。** Claude Code はリポジトリ直下の `skills/` と `agents/` を見る。`.apm/` へ移すと `plugin.json` の `skills` / `agents` でパスを指定し直す必要がある。
2. **`agents` はディレクトリを受け付けない。** `"agents": ["./.apm/agents/"]` は `agents: Invalid input` で検証が失敗する。ファイルパスの列挙しかできないため、エージェントを追加するたび `plugin.json` を編集することになる。
3. **`*.agent.md` が Claude Code で読めるか未検証である。**
4. **移行しなくても APM から使える。** APM は `.apm/` も `apm.yml` も無い素の `skills/` リポジトリを install できる。上記の先例が実証している。

## 却下: `apm.yml` の追加

`apm.yml` は「このパッケージが何に依存するか」を宣言するファイルである。satomi-skills は他の primitive に依存しないため、置いても空の宣言になる。

`core` スキルの「要求されていない柔軟性や設定可能性を加えない」に反する。**将来、他のスキルパッケージに依存するようになったときに初めて追加する。**

## 検証済みの制約

構成を変えるときに踏みやすい落とし穴。すべて Claude Code 2.1.228 で実測した。

| やること | 結果 |
|---|---|
| `agents` にディレクトリを渡す | `claude plugin validate` が `agents: Invalid input` で失敗 |
| `skills/<name>/agent.md` を `agents` に列挙 | **validate は通るが実行時にロードされない**（`Agents (0)`）。最も危険 |
| `agents/<name>.md` に置く | 正常にロードされる |
| `.claude-plugin/` に plugin.json 以外を置く | marketplace.json のみ許容。他は全てリポジトリ直下 |
| プラグインの `settings.json` | `agent` と `subagentStatusLine` しか受け付けない。ユーザ設定は配布できない |

## 他クライアントへの配布

`skills/<name>/` をディレクトリごとコピーすれば動く。書き換えは不要。

`agents/` は Claude Code 固有のため他クライアントでは読み込まれない。スキル単体でも規約の出典として機能するよう、**手順と規約はスキル側に置く**という原則がここでも効く。

## 出典

- [Agent Skills 仕様](https://agentskills.io/specification)
- [APM (Agent Package Manager)](https://github.com/microsoft/apm)
- [Claude Code — Plugins reference](https://code.claude.com/docs/en/plugins-reference)
