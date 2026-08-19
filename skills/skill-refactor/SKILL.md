---
name: skill-refactor
description: 既存のスキルが長くなりすぎたときに使う。on-invoke トークンが予算を超えたスキルを計測し、毎回必要な内容と必要になったときだけ読む内容に振り分けて references/ へ分割し、索引を張って検証する手順を定める。スキルを分割する、スキルを軽くする、progressive disclosure を効かせる、トークンコストを下げるときに参照する。
allowed-tools: Bash(claude plugin details *) Bash(claude plugin validate *) Bash(awk *) Bash(wc *) Bash(grep *)
metadata:
  paired-agent: satomi-skills:skill-refactor
---

# スキル分割手順

長くなったスキルを progressive disclosure で軽くする手順を示す。新規作成は `skill-builder` が担当する。

## 2 種類のコスト

| 種類 | 何が乗るか | いつ払うか |
|---|---|---|
| always-on | `name` と `description` | **全セッション**。スキルを使わなくても乗る |
| on-invoke | `SKILL.md` の本文全体 | スキルが起動するたび |

`references/` 配下のファイルはどちらにも乗らない。**本文から参照され、実際に読まれたときだけ**払う。分割はこの性質を使ってコストを移す作業である。

## 予算

| 対象 | 基準 | 出典 |
|---|---|---|
| `SKILL.md` 本文 | 5,000 トークン以内 | Agent Skills 仕様の推奨 |
| `SKILL.md` 本文 | 500 行以内 | 同上 |
| リファレンス参照の深さ | `SKILL.md` から 1 階層 | 同上 |
| エージェント本文 | 1 段落・3 文以内 | このリポジトリの規約 |

日本語の本文では概ね 1 行あたり 19 トークン前後（実測からの概算）。260 行を超えたあたりから 5,000 に近づく。**行数は目安にしかならないので必ず計測する。**

## 着手の判断

次のいずれかに当てはまったら分割する。

- `on-invoke` が 5,000 トークンを超えている。
- 本文が 500 行を超えている。
- 節が 10 を超え、冒頭に索引が無い。
- 特定の出力先・特定の条件でしか使わない節が本文の 3 割以上を占めている。

## 手順

### 1. 計測する

```
claude plugin details satomi-skills
```

`Per-component` の `on-invoke` 列を見る。スキルとエージェントが同名で 2 行ずつ並ぶので、上のブロックがスキル、下がエージェントである。

### 2. 節ごとの行数を出す

```
awk '/^#{2,3} /{if(h){printf "%-46s %4d 行\n", h, NR-s} h=$0; s=NR} END{if(h) printf "%-46s %4d 行\n", h, NR-s}' skills/<name>/SKILL.md
```

`###` まで出すと、大きい節の中のどこが重いか分かる。節単位で切れない場合は小見出し単位で切る。

### 3. 残すものと出すものを振り分ける

判断軸は**毎回必要か、必要になったときだけ読めばよいか**の一点。

| `SKILL.md` に残す | `references/` に出す |
|---|---|
| 適用手順、原則、判断基準 | 網羅的な一覧表・対応表 |
| 使うたびに効かせたい規約 | 記法や属性の細目 |
| やらないこと | 背景・理由・実測記録 |
| 参照先の索引 | 特定の出力先・条件でだけ必要な実装詳細 |

**判断基準は残し、実装詳細を出す。** 「タブをいつ使うか」は残し、「タブの HTML マークアップ」は出す。

### 4. 分割する

1. `skills/<name>/references/<topic>.md` を作る。1 ファイル 1 主題に絞る。
2. 冒頭に何のリファレンスかを 1〜2 文で書き、`SKILL.md` に何があるかを示す。
3. 切り出した節を移す。**見出し階層を詰める**（`####` が `##` の直下に来ないよう繰り上げる）。
4. `SKILL.md` の切り出し跡にリファレンスへのポインタを 1 行残す。
5. `SKILL.md` 冒頭に参照索引を表で置く。
6. **出典を移す。** 移した内容の根拠となる外部リンクはリファレンス側に付け替える。`SKILL.md` に残る内容の根拠が無くなったら `## 出典` 自体を畳む。

### 5. 検証する

```
claude plugin validate .claude-plugin/plugin.json --strict
claude plugin details satomi-skills
```

| 項目 | 確認方法 |
|---|---|
| 部品数が変わっていない | `Skills (n)` / `Agents (n)` が分割前と同じ |
| 予算に収まった | `on-invoke` が 5,000 トークン以内 |
| 参照先が実在する | `grep -o 'references/[a-z-]*\.md' skills/<name>/SKILL.md` の各件がファイルとして存在する |
| 連鎖していない | `grep -l 'references/' skills/<name>/references/*.md` が空、または該当箇所がディレクトリ名の言及にとどまる |
| 見出し階層が飛んでいない | `grep -n '^#' skills/<name>/references/*.md` |

## 実例 — document-design

340 行 / on-invoke 約 7.3k トークンを分割した記録。

| | 分割前 | 分割後 |
|---|---|---|
| `SKILL.md` 行数 | 340 | 161 |
| `SKILL.md` on-invoke | 約 7.3k tok | 約 2.7k tok |
| リファレンス | なし | `skills/document-design/references/html-implementation.md` 194 行 |

出したもの: マークアップの語彙、書体、カラートークン、レイアウト、React Aria 互換の命名、Artifact の制約、印刷 CSS、タブとアコーディオンの HTML。すべて **HTML 出力のときだけ必要**な実装詳細である。

残したもの: 適用手順、出力先ごとの扱い、文章の規則、動的な表現の判断基準、やらないこと。**出力先を問わず毎回効く内容**である。

外部リンク 6 件（React Aria / WCAG / iOS カラー / `::details-content`）はすべて出した内容の根拠だったため、リファレンス側へ移して `SKILL.md` の `## 出典` を畳んだ。

## やらないこと

- **要点をリファレンスに隠す。** `SKILL.md` だけ読んで作業が成立する状態を保つ。リファレンスは深掘り用にする。
- リファレンスから別のリファレンスを参照する。1 階層を超えると読まれない。
- 1 ファイルに複数の主題を詰める。読み込みコストが下がらない。
- 索引を置かずに分割する。何がどこにあるか分からなくなる。
- 切り出し跡にポインタを残さない。本文だけ読むと規約が欠けているように見える。
- 出典を移さずに放置する。根拠と主張が別ファイルに分かれる。
- 計測せずに行数だけで判断する。
- 分割と同時に内容を書き換える。まず移すだけにして、検証が通ってから直す。

## eval による回帰確認

分割で挙動が劣化していないかを測る仕組みが用意されている。

```
skills/<name>/evals/<case>/case.yaml     # または prompt.md
skills/<name>/evals/<case>/graders/*.md
```

```
claude plugin eval satomi-skills
```

プラグイン名で指定するとスキル無しの baseline と比較したスコア差分も出る。判定モデルは既定で haiku、`--max-cost-usd` で上限を設けられる。

このリポジトリには未導入である。記述を大きく変えるときに導入を検討する。

## 出典

- [Agent Skills 仕様 — Progressive disclosure](https://agentskills.io/specification)
- [Claude Code — Skills](https://code.claude.com/docs/en/skills)
