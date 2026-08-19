---
name: skill-builder
description: このリポジトリにスキルを追加する、既存のスキルを修正する、対応するエージェントを用意する、スキルの書式や配置を確認するときに使う。satomi-skills の配置規約、Agent Skills 標準に収める frontmatter の書き方、1 スキル 1 エージェントの対応付け、トークン予算に収める分割方針、検証と反映の手順を定める。
license: MIT
allowed-tools: Bash(claude plugin validate *) Bash(claude plugin details *) Bash(git status *) Bash(git diff *)
metadata:
  paired-agent: satomi-skills:skill-builder
---

# スキル作成手順

satomi-skills にスキルとエージェントを追加・変更するための配置規約と書式を示す。

詳細は必要になったときに読む。

| 参照先 | 内容 |
|---|---|
| `references/frontmatter.md` | スキルとエージェントの全フィールド仕様、`allowed-tools` の記法、標準に絞る理由 |
| `references/compatibility.md` | 他クライアントと APM の互換性。構成を変えたくなったときに先に読む |

長くなったスキルの計測と分割は `skill-refactor` が担当する。このスキルは新規作成を担当する。

## リポジトリの構成

```
satomi-skills/
├── .claude-plugin/
│   ├── plugin.json          # プラグイン定義（name: satomi-skills）
│   └── marketplace.json     # 配布定義（source: "./" で自己参照）
├── skills/<name>/SKILL.md   # スキル本体。同梱ファイルは同ディレクトリに置く
├── agents/<name>.md         # スキルに対応するエージェント
└── README.md
```

- スキルは `skills/<name>/SKILL.md`。呼称は `/satomi-skills:<name>`。
- エージェントは `agents/<name>.md`。呼称は `@satomi-skills:<name>`。
- ディレクトリ名は kebab-case。目的を表す名詞句か動詞句にする（例 `commit`, `document-design`）。
- `.claude-plugin/` に置くのは `plugin.json` と `marketplace.json` だけ。他のディレクトリはすべてリポジトリ直下に置く。

## 原則 — 1 スキル 1 エージェント

スキルを追加したら、対応するエージェントを必ず 1 つ追加する。

| 要素 | 置き場所 |
|---|---|
| 手順・規約・禁止事項 | スキル本文 |
| ペルソナ | エージェント本文（1 段落・3 文以内） |
| スキルとの対応付け | エージェントの `skills:` |
| 対応の逆引き | スキルの `metadata.paired-agent` |
| 担当範囲 | エージェントの `description` と `tools` |

**同じ規約を 2 箇所に書かない。** 片方だけ古くなる。スキルが「何をどうするか」、エージェントが「誰であるか」で分ける。

対応関係を確認する。

```
grep -r "paired-agent" skills/
grep -rA3 "^skills:" agents/
```

### core スキルは全エージェントにプリロードする

`core` は作業全体の基準を持つスキルである。**新しいエージェントを作るときは `skills:` の先頭に `satomi-skills:core` を必ず入れる。**

スキル本文は起動時に常時読み込まれるわけではなく、呼び出されたときに読み込まれる。常に効かせたい基準はプリロードで担保する。

## 最小書式

### スキル

```markdown
---
name: <ディレクトリ名と同じ>
description: 何をするか + いつ使うか。自動起動の判断キーになるため具体的に書く
license: MIT
metadata:
  paired-agent: satomi-skills:<ディレクトリ名と同じ>
---

# 見出し

本文。手順や規約を書く。
```

`SKILL.md` の frontmatter は **Agent Skills 標準の 6 フィールド（`name` `description` `allowed-tools` `metadata` `license` `compatibility`）だけを使う。** 標準外を入れると claude.ai / Skills API へのパッケージングがハードエラーで失敗する。詳細と代替手段は `references/frontmatter.md`。

### エージェント

```markdown
---
name: <スキル名と同じ>
description: 何を担当するか。いつ委譲すべきか
skills:
  - satomi-skills:core
  - satomi-skills:<スキル名>
tools: Bash, Read, Grep, Glob
color: green
---

あなたは（何者か）である。（何をしない人か / どう振る舞うか）。
```

本文はペルソナだけ。担当範囲が振る舞いとして重要な場合は、規約として並べずペルソナの言い回しに畳む。「担当はコミットを積むことだけ。実装は変更しない」ではなく「動くコードを書く人ではない」と書く。

### 副作用のあるスキルの扱い

`disable-model-invocation: true` は使わない。標準外フィールドである上に、**この指定が付いたスキルはエージェントにプリロードできない。** 1 スキル 1 エージェントの構成と両立しない。

自動起動を避けたい副作用は、スキル本文に確認手順を書いて制御する。

## 同梱ファイル

Agent Skills 仕様の慣例に従う。

| ディレクトリ | 用途 |
|---|---|
| `references/` | 追加のドキュメント。必要になったときに読む |
| `scripts/` | 実行可能なコード。依存を自己完結させるか明記する |
| `assets/` | テンプレート、画像、データファイル |

- 参照は `SKILL.md` からの相対パスで書く（`references/<name>.md` の形）。
- 1 階層より深くしない。リファレンスから別のリファレンスへ連鎖させない。
- 1 ファイルを 1 主題に絞る。

## 追加手順

1. `skills/<name>/` を作り、`SKILL.md` を最小書式で書く。
2. `agents/<name>.md` を作り、`skills:` に `satomi-skills:core` と `satomi-skills:<name>` を書く。
3. 補助ファイルは `skills/<name>/` 内に置き、`SKILL.md` から相対パスで参照する。
4. 検証する（次節）。
5. README.md のスキル一覧に 1 行足す。

## 検証

マニフェストを検証する。`--strict` は警告もエラーとして扱う。

```
claude plugin validate .claude-plugin/plugin.json --strict
claude plugin validate .claude-plugin/marketplace.json --strict
```

**マニフェストの検証だけでは足りない。** 実際にロードされたかを部品数で確認する。

```
claude plugin details satomi-skills
```

確認する項目。

| 項目 | 基準 |
|---|---|
| `Skills (n)` | `skills/` の実数と一致 |
| `Agents (n)` | `agents/` の実数と一致 |
| スキルの `on-invoke` | 5,000 トークン以内。超えたら `/satomi-skills:skill-refactor` で分割する |
| `Always-on` | 増やしすぎない。全セッションに毎回乗る |

手で確認する項目。

- `SKILL.md` の `name` が親ディレクトリ名と一致している。
- `SKILL.md` の frontmatter が 6 フィールドに収まっている。
- スキルとエージェントが 1 対 1 で、`skills:` に `core` が入っている。

## 反映

- ローカルの編集: Claude Code はスキルとエージェントのディレクトリを監視する。編集は数秒で反映され、再起動は不要。
- 新しいスコープに初めて `agents/` を作った場合は再起動が必要。
- リモートから入れている場合: `/plugin marketplace update satomi-skills`。

## やらないこと

配置:

- **`skills/<name>/` の中にエージェントを置く。** `plugin.json` の `agents` にそのパスを列挙すると `claude plugin validate` は通るが、**実行時にロードされない**（`Agents (0)` になる）。検証をすり抜けるため最も危険な壊れ方をする。エージェントは `agents/` 直下に置く。
- **`agents` にディレクトリを渡す。** `"agents": ["./skills/"]` は `agents: Invalid input` で検証が失敗する。`agents` は `.md` ファイルのパスしか受け付けない。
- **`.apm/` 配下へ移す、`apm.yml` を置く。** どちらも不要かつ有害である。判断の根拠は `references/compatibility.md` に記録した。**構成を変えたくなったら先にそれを読む。**

内容:

- 標準外の frontmatter フィールドを `SKILL.md` に書く（パッケージングが失敗する）。
- `name` を親ディレクトリ名と違える（Agent Skills 仕様違反）。
- **エージェント本文に手順や規約を書く。** それはスキルの担当である。
- エージェント本文で `skills:` の内容を散文に書き直す。frontmatter が宣言済みである。
- 長大なリファレンス全文を `SKILL.md` に入れる。`references/` に分け、`SKILL.md` は索引にする（手順は `skill-refactor`）。
- 作業全体に効かせたい方針を個別スキルに書く。それは `core` スキルに書き、各エージェントへプリロードする。
- `core` スキルにある内容を個別スキルへ重複させる。
- スキルを追加してエージェントを用意しない。
- 検証を飛ばして完了と報告する。

## 出典

- [Agent Skills 仕様](https://agentskills.io/specification)
- [Claude Code — Skills](https://code.claude.com/docs/en/skills)
- [Claude Code — Subagents](https://code.claude.com/docs/en/sub-agents)
- [Claude Code — Plugins reference](https://code.claude.com/docs/en/plugins-reference)
- [Claude Code — Plugin marketplaces](https://code.claude.com/docs/en/plugin-marketplaces)
- [Claude Code — Permissions](https://code.claude.com/docs/en/permissions)
