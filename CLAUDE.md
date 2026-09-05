# public_website

GitHub Pages で公開している静的 HTML ページ群。各ページは外部依存を持たないスタンドアロン HTML（CSS・JS をファイル内にインライン）で書く。

## デザイン原則

このリポジトリの全ページに適用する。新規ページの作成時も、既存ページの修正時も、この原則から外れる実装をしない。

### 1. グラデーションを使わない

`linear-gradient` / `radial-gradient` を装飾目的で使わない。面は単色で塗る。奥行きは色の明度差と 1px の罫線で表現する。

- カバー・ヒーロー・カード・ボタン・ヘッダーの背景はすべて単色
- 装飾以外の用途（セレクトの矢印など CSS で図形を描く場合）は使わず、インライン SVG の `data:` URI にする

### 2. 絵文字をアイコンとして使わない

絵文字は本文コンテンツ（作品タイトルなど原文に含まれる文字）にのみ許可する。UI 要素としては使わない。

- favicon に絵文字を埋め込まない。単色の SVG マークにする
- 見出し・ボタン・ラベル・リストマーカーに絵文字を付けない
- アイコンが必要な場合は `currentColor` で塗るインライン SVG（stroke 1.5px、24px グリッド）
- `✕` `→` `☆` などの幾何記号・約物は絵文字ではないので、UI ラベルに使ってよい

### 3. 余白を広く取る

情報を詰めない。要素の間隔は 4px の倍数（実質 8px グリッド）に揃える。

| 用途 | 目安 |
| --- | --- |
| インライン要素・ラベル間 | 4 / 8px |
| カード内のブロック間 | 12 / 16px |
| カードのパディング | 20〜24px |
| カード同士の間隔 | 12 / 16px |
| セクション間 | 48〜64px |
| ページ左右のパディング | 24px（モバイル）／ 40px（デスクトップ） |
| 本文の最大幅 | 720〜880px |

行間は本文 1.7〜1.8、見出し 1.3 前後。文字を詰めるより、まず余白を削らないことを優先する。

### 4. モノクロ基調 + アクセント1色

配色はニュートラルなグレースケールを土台にし、彩度のある色はアクセント1色だけ使う。アクセントはリンク・選択状態・強調ラベルなど「操作できる／今ここ」を示す箇所に限る。

標準トークン（`index.html` / `kakeibo-analytics.html` が基準）:

```css
:root {
  color-scheme: light;
  --page-bg:        #f9fafb;
  --surface:        #ffffff;
  --border:         #e4e7ec;
  --text-primary:   #101828;
  --text-secondary: #667085;
  --brand:          #465fff;  /* アクセント。1色だけ */
  --brand-tint:     #ecf3ff;
  --shadow-card:    0 1px 3px rgba(16,24,40,0.06), 0 1px 2px rgba(16,24,40,0.04);
}
@media (prefers-color-scheme: dark) {
  :root {
    color-scheme: dark;
    --page-bg:        #101828;
    --surface:        #1a2231;
    --border:         #1d2939;
    --text-primary:   #f9fafb;
    --text-secondary: #98a2b3;
    --brand:          #7592ff;
    --brand-tint:     rgba(70,95,255,0.14);
    --shadow-card:    0 1px 3px rgba(0,0,0,0.3), 0 1px 2px rgba(0,0,0,0.24);
  }
}
```

一覧系ページ（`favorite-*.html` / `gunpla-gallery.html`）は同じ考え方の暖色ニュートラル版を使っている。こちらもモノクロ基調で、アクセントは `--mark`（地色の反転色）1つだけ。既存ページを触るときは、そのページが使っている方のトークンに合わせる。

```css
:root {
  --bg: #eceae5;  --bg-sub: #dedbd4;  --panel: #f6f5f2;
  --text: #17171a; --text-muted: #7d7c76;
  --rule: #d2cfc8; --rule-strong: #17171a;
  --mark: #17171a; --on-mark: #eceae5;
}
```

例外的に多色が要るのはデータ可視化のみ（`kakeibo-analytics.html` の系列色）。グラフの系列以外に多色を持ち込まない。

### 5. 罫線と影は控えめに

- 境界は `1px solid var(--border)`。2px 以上の太い枠線を使わない
- 影は `--shadow-card` 相当の 1〜3px の弱い影まで。大きなドロップシャドウで浮かせない
- 角丸は 8px（小要素）／ 12〜16px（カード・パネル）に統一する

### 6. その他の共通事項

- ライト／ダーク両対応。色は必ず `:root` の変数に定義し、`prefers-color-scheme: dark` で上書きする
- フォントは端末標準を使う。`-apple-system, BlinkMacSystemFont, "Hiragino Kaku Gothic ProN", "Hiragino Sans", "Yu Gothic", "Segoe UI", sans-serif`
- レスポンシブ。横スクロールが必要な表・グラフは、その要素だけ `overflow-x: auto` の中に入れる
- 外部 CDN の CSS/JS に依存しない

## 変更したら確認すること

HTML を追加・変更したら push 前に次を確認する。

```bash
grep -n "linear-gradient\|radial-gradient" *.html   # 0 件であること（データ可視化を除く）
grep -n 'rel="icon"' *.html                          # favicon に絵文字が埋まっていないこと

# 絵文字の検出。HTML 実体参照（&#128077; など）も展開して見る
python3 -c "
import glob, re, html
for f in sorted(glob.glob('*.html')):
    s = open(f, encoding='utf-8').read()
    for m in re.finditer(r'&#x?[0-9A-Fa-f]+;|.', s):
        d = html.unescape(m.group(0))
        if len(d) == 1 and ord(d) > 0x1F000:
            print(f, repr(m.group(0)))
"
```

`&#128077;` のような実体参照は素の `grep` に引っかからないので、必ず展開してから確認する。

## 公開時のルール

新規ページを公開するときは、同じ push で `index.html` にリンクカードを追加する。既にリンク済みのページの中身を更新するだけの作業では `index.html` を触らない。
