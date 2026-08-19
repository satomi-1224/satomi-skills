---
name: commit
description: コミットを作成する、コミットメッセージを起案する、コミットの粒度や分割を判断するときに使う。type は feat/fix/docs/style/refactor/perf/test/chore に限り、1 コミット = 1 type に収める分割規則と、認証情報を含めない禁止事項を定める。
allowed-tools: Bash(git status *) Bash(git diff *) Bash(git log *) Bash(git add *) Bash(git commit *)
metadata:
  paired-agent: satomi-skills:commit
---

# コミット規約
コミット時の粒度とルールを示す

## メッセージ書式

```
<type>: <subject>
```

- `subject` は変更内容を簡潔な日本語で書く
- 以下の表に無いtypeは使わない

## type 一覧

| type | 用途 |
|---|---|
| feat | 新機能の追加 |
| fix | バグ修正 |
| docs | ドキュメントのみの変更 |
| style | コードの意味に影響を与えない変更（空白、フォーマット、セミコロンなど） |
| refactor | バグ修正も機能追加も行わないコードの改善 |
| perf | パフォーマンス向上関連 |
| test | テストコード関連 |
| chore | ビルド、補助ツール、ライブラリの更新など |

## ルール

- コミットはファイル単位ではなく変更単位で行うこと
- 責務が違うコミットを混ぜないこと
- 1 コミット = 1 type に収めること。複数 type に跨る変更は type ごとに分割すること
- 実装変更とドキュメント更新（README / docs / コメント）は別コミットにすること（type が `feat`/`fix` 等と `docs` で異なるため）
- 機能追加とリファクタリング、機能追加とテスト追加もそれぞれ別コミットにすること
- フォーマット変更が混ざる場合は `style` として別コミットに分離すること
- 設定変更（CI / build / dependency / nix 等）と機能変更は別コミットにすること（前者は `chore`）
- コミット前にステージ内容を `git status` / `git diff --staged` で確認し、type が複数想起されたら分割すること
- 分割可否が判断できないときはユーザに確認してから進めること
- `.env` / 認証情報を含むコミットは行わないこと
- pre-commit hook が失敗した場合は `--no-verify` でスキップせず、原因を修正してから新規コミットを作成すること
- `--amend` / force push は明示要求があった場合のみ実行すること
