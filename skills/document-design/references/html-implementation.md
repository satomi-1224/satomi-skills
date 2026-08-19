# HTML 実装リファレンス

HTML 出力（Artifact / 単体 HTML）のときだけ必要になる実装規約。
文章の規則と適用手順は `SKILL.md` にある。

## 動的な表現のマークアップ

### アコーディオン

```html
<details open>
  <summary>詳細ログ</summary>
  <div class="details-body">…</div>
</details>
```

- **印刷を想定する資料では `open` を既定で付ける。**
  閉じた `details` を CSS で開けるのは `::details-content` 対応ブラウザ
  （Chrome 131+ / Firefox 143+ / Safari 18.4+）に限られ、それ以前では中身が印刷されない。
  子要素の `display` を復元しても開かない（UA が内部の `content-visibility` で隠すため）
- 閉じて配布したい場合のみ `open` を外す。その場合、旧ブラウザの印刷では中身が消えることを許容する

### タブ

```html
<div class="tabs">
  <input class="tab-input" type="radio" name="env" id="env-a" checked>
  <label class="tab-label" for="env-a">macOS</label>
  <input class="tab-input" type="radio" name="env" id="env-b">
  <label class="tab-label" for="env-b">Linux</label>
  <div class="tab-panels">
    <section class="tab-panel" data-title="macOS">…</section>
    <section class="tab-panel" data-title="Linux">…</section>
  </div>
</div>
```

- **`data-title` を必ず付ける。** 印刷ではラベルが消え、全パネルが `data-title` の見出し付きで並ぶ
- radio は非表示だがフォーカス可能（visually hidden）。矢印キーで切り替えられる
- パネルの表示切り替えは nth 規則で対応付ける。`reference.css` は 6 タブまで定義済み。
  超える場合は同じ形の規則を足す

## マークアップの語彙

`reference.css` と印刷 CSS はこの語彙を前提に書かれている。**別名を使うと印刷対応が効かない。**

| クラス | 役割 |
|---|---|
| `.wrap` | ページコンテナ。`position: relative` |
| `.doc-meta` | 作成日時スタンプ |
| `.group` / `.row` | グループ化リストとその行 |
| `.tscroll` | 表・図の横スクロール容器 |
| `.tabs` `.tab-input` `.tab-label` `.tab-panels` `.tab-panel[data-title]` | タブ一式 |
| `.details-body` | アコーディオンの中身 |
| `.chip` | 小型ラベル |
| `.status-success` / `.status-warning` / `.status-danger` | 意味色の状態テキスト |

## 書体

和文ゴシックのシステムフォント。Web フォントを使わない。

- スタックは `reference.css` に定義済み。先頭は "Hiragino Sans"
  （"Hiragino Kaku Gothic ProN" よりウェイト展開が広く、400 / 500 / 600 の 3 段を描き分けられる）
- **Noto Sans JP / Yu Gothic / Meiryo をフォールバックに含める。**
  Artifact は共有されるため、macOS 以外の閲覧者を前提から外さない
- 欧文専用書体は併記せず、英数字も和文ゴシックに任せる
- 等幅のみ例外: `ui-monospace, "SF Mono", Menlo, Consolas, monospace`
- ウェイトは 400（本文）/ 500（強調・選択状態）/ 600（見出し）の 3 段のみ。700 は使わない
- 見出しには `letter-spacing: -.01em` と `text-wrap: balance`
- 本文の行長は**全角 40〜45 文字**。`reference.css` の `.wrap`（max-width 760px）がこの幅になる。
  `ch` 単位で指定しない（`ch` は半角 "0" の幅で、全角文字数の目安にならない）

## カラートークン — Warm Graphite

暖色に寄せたグレーが階層を担い、アクセントは無彩色。**色相を 1 つに保つ。**
値は `reference.css` にあり、`prefers-color-scheme` と `data-theme` の両方で上書き済み
（トグルが media query に勝つ必要がある）。

| トークン | 役割 |
|---|---|
| `--bg` / `--surface` / `--raised` | ページ地 / カード・グループ / その上の一段 |
| `--sep` | 罫線 |
| `--label` / `--sec` / `--ter` | 本文 / 2 次テキスト / 3 次テキスト |
| `--accent` / `--accent-fg` | 操作可能な要素 / その上の文字 |
| `--success` / `--warning` / `--danger` | 意味色 |
| `--shadow` | 明テーマのみ。暗テーマは `none` |

### 守ること

- **コンポーネントはトークン経由でのみ色を参照する。** media query の内側に
  コンポーネントのスタイルを書かない（トークンだけ再定義する）
- **アクセントは操作できるものにだけ与える。** 装飾に使わない
- **`--sep` の対比は 1.1〜1.9 で意図的に低い。** 罫線であって文字ではない。
  区切りは色の強さでなく `gap` と余白で作る
- **意味色は accent と別枠。** success / warning / danger を強調目的で使わない
- 影は明テーマのみ。暗テーマは `--surface` → `--raised` の明度差で階層を出す

### コントラスト実測値

WCAG 2.x の相対輝度式で計算済み。**本文・意味色は全サーフェスで AA 4.5 以上。**

| | 明 `--bg` / `--surface` / `--raised` | 暗 `--bg` / `--surface` / `--raised` |
|---|---|---|
| `--label` | 15.02 / 16.66 / 15.83 | 19.44 / 15.59 / 12.89 |
| `--sec` | 5.93 / 6.58 / 6.25 | 8.96 / 7.19 / 5.94 |
| `--ter` | 4.53 / 5.02 / 4.77 | 7.30 / 5.86 / 4.84 |
| `--success` | 5.84 / 6.47 / 6.15 | 11.52 / 9.24 / 7.64 |
| `--warning` | 6.18 / 6.85 / 6.51 | 11.86 / 9.52 / 7.86 |
| `--danger` | 6.81 / 7.55 / 7.18 | 8.61 / 6.91 / 5.71 |
| `--accent` / `--accent-fg` | 13.14 | 13.28 |

トークンを変えるのは `reference.css` のみ。変えたらこの表を再計算する。
`--ter` は明 `--bg` 上で 4.53 と余裕がないので、これ以上薄くしない。

## レイアウト

iOS のグループ化リストを骨格にする。

- 角丸: グループ 12px / カード内 9px / ボタン 8px / チップ 5px
- グループは `background: var(--surface)` + `border-radius: 12px` + `box-shadow: var(--shadow)`
- **行間の罫線は `border` ではなく `box-shadow: inset 0 .5px 0 var(--sep)`**（0.5px を出すため）
- 兄弟要素の間隔は flex/grid の `gap`。個別の `margin` で作らない
- 横に溢れるもの（表・コード・図）は `.tscroll` に入れる。body を横スクロールさせない
- 数字が縦に並ぶ箇所は `font-variant-numeric: tabular-nums`（表には適用済み）
- セレクタの詳細度に注意する。型と要素の指定が余白を打ち消し合う事故が起きやすい

## コンポーネント — React Aria 互換の命名

**HTML 出力（Artifact / 単体 HTML）の場合のみ適用。**

静的ドキュメントに React Aria 本体は載せられない（Artifact は外部ホスト不可、
単体 HTML もビルドが無い）。そこで素の HTML/CSS を
<https://react-aria.adobe.com/> のクラス名・データ属性の語彙で書き、後から載せ替え可能にする。

- クラス名は React Aria の既定に合わせる: `.react-aria-Button`, `.react-aria-Switch`,
  `.react-aria-Checkbox`, `.react-aria-Slider`, `.react-aria-ListBox`,
  `.react-aria-ListBoxItem`, `.react-aria-Select`, `.react-aria-Dialog` など
- **静的出力ではデータ属性（`[data-hovered]` 等）は自動では付かない。**
  React Aria の JS が実行時に付けるものなので、データ属性だけに書くと hover も press も一切効かない
- したがって状態は**擬似クラスとデータ属性の併記**で書く。
  `:hover` は `@media (hover: hover)` で包み、タッチ端末の張り付きを防ぐ
  （React Aria 移植後は `[data-hovered]` 側がそのまま効く）。`reference.css` がこの形になっている
- フォーカスは `outline: 2.5px solid var(--accent); outline-offset: 2px`
- ホバーは色を変えるより `opacity: .88`、押下は `.7` が最も破綻しない
- 無効は `opacity: .35` + `cursor: default`
- `prefers-reduced-motion: reduce` で transition を切る（`reference.css` に定義済み）

トグルは `.react-aria-Switch`（幅 44 / 高さ 26 / つまみ 22px、
`transform: translateX(18px)`、`cubic-bezier(.32,.72,0,1)`）。CSS は `reference.css` にある。

## 出力先が Artifact の場合

- CSP により外部ホストへの通信が全て不可。CSS/JS はインライン、画像は data URI
- **Web フォントの URL 参照は不可**（無言でフォールバックする）。システムフォントで組む
- `<!doctype>` `<html>` `<head>` `<body>` は書かない。公開時に包まれる
- `<title>` を必ず入れる。再公開しても変えない
- favicon は絵文字で渡す。再公開時に変えない

## 印刷 CSS（HTML 出力では必須）

**HTML を書くときは常に `@media print` を含める。** 印刷が想定用途に無くても入れる。
資料はいつ印刷されるか分からず、崩れた状態で印刷されるのを防げないため。
`reference.css` 末尾のブロックをそのまま使い、**スタイルシートの最後に置く。**

明テーマへの固定は `:root, :root[data-theme="dark"]` の 2 セレクタで上書きしている。
`:root` 単独 (0,1,0) では `:root[data-theme="dark"]` (0,2,0) に詳細度で負け、
**暗テーマにトグルしたユーザの印刷が真っ黒になる。** セレクタを削らない。

### 印刷で崩れる原因と対処

| 原因 | 症状 | 対処（`reference.css` に定義済み） |
|---|---|---|
| 暗テーマのまま | 地が真っ黒でインクを大量消費 | 明テーマ固定。`data-theme` 側のセレクタも上書き |
| `overflow-x: auto` | 表・コードの右側が切れる | `overflow: visible` + `min-width: 0` に戻す |
| `min-width` 付きの表 | 紙幅を超えて切れる | `width: 100%` を強制 |
| `box-shadow` で領域表現 | 境界が消えて構造が読めない | 罫線に置換 |
| 影・薄い罫線 | 印刷に出ない | `--sep` を濃い値に差し替え |
| リンク | URL が分からない | `::after` で URL を併記 |
| 表が改ページを跨ぐ | 見出し行が消える | `thead { display: table-header-group }` |
| 閉じたアコーディオン | 中身が印刷されない | `open` を既定にする。CSS での復元は `::details-content` 対応ブラウザ限定 |
| タブ | 非表示パネルが印刷されない | 全パネルを展開し `data-title` の見出しを付ける |
| `100vh` 指定 | 空白ページが混入 | 印刷時に高さ指定を解除（使った場合は自分で解除を書く） |

`100vh` や `position: fixed` は印刷で空白ページや要素の重複を生む。
画面用に使った場合は `@media print` で必ず解除する。

## 出典

- React Aria: <https://react-aria.adobe.com/>
- React Aria スタイリング（データ属性の一覧）: <https://react-aria.adobe.com/styling.html>
- WCAG 2.2 コントラスト最低基準: <https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum.html>
- iOS のカラー設計（グレー階段 + 単色アクセントの発想元）: <https://developer.apple.com/design/human-interface-guidelines/color>
- `::details-content` 擬似要素: <https://developer.mozilla.org/en-US/docs/Web/CSS/::details-content>
- `::details-content` ブラウザ対応状況: <https://caniuse.com/mdn-css_selectors_details-content>
