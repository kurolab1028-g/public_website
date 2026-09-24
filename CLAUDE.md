# public_website

kurolab1028-g の個人サイト。素の HTML/CSS/JS のみで構成し、ビルド処理はない。
GitHub Pages が `main` ブランチのルートをそのまま配信している
（公開URL: https://kurolab1028-g.github.io/public_website/ ）。

## 作品一覧ページ

| ページ | データ配列 | 画像ディレクトリ |
|---|---|---|
| `favorite-anime.html` | `ANIME_DATA` | `images/favorite-anime/` |
| `favorite-manga.html` | `MANGA_DATA` | `images/favorite-manga/` |
| `favorite-movie.html` | `MOVIE_DATA` | `images/favorite-movie/` |

各ページはファイル内の `<script>` に定義された配列だけがデータソース。
配列は追加日（`addedDate`）の新しい順に並べ、新規作品は配列の先頭に挿入する。

## 作品追加の手順（アニメ・漫画・映画）

「アニメ／漫画／映画に○○を追加して」と言われたら、**公開までを一続きの作業として完了させる**。
途中で止めず、最後に公開URLで反映を確認するところまでを毎回行う。

1. **情報を調べる** — 対象ページの既存エントリとまったく同じキー構成・表記ゆれで埋める。
   巻数・話数や連載状況は公式サイト等で最新の値を確認する。
2. **画像を取得する** — 書影・ポスターは Amazon の
   `https://images-na.ssl-images-amazon.com/images/P/<ASIN>.09.LZZZZZZZ.jpg` から取得し
   （既存画像と同じ 351x500 程度になる）、`images/<ページ名>/<id>.jpg` として保存する。
   画像が無いエントリを作らない。
3. **配列に追加する** — 配列の先頭に1行で挿入し、`addedDate` は当日の日付にする。
4. **検証する** — データ配列を Node でパースし、構文エラー・ID重複・画像ファイル欠落がないことを確認する。

   ```bash
   sed -n '/^const MANGA_DATA = \[/,/^\];/p' favorite-manga.html > /tmp/d.js
   node -e "const fs=require('fs');
   const M=new Function(fs.readFileSync('/tmp/d.js','utf8')+'; return MANGA_DATA;')();
   console.log(M.length, M.length-new Set(M.map(m=>m.id)).size,
               M.filter(m=>!fs.existsSync(m.image)).map(m=>m.id));"
   ```

5. **作業ブランチにコミット & プッシュする**。
6. **`main` にマージしてプッシュする** — ここまでやらないと公開されない。
   `main` が進んでいてコンフリクトした場合は、**どちらのエントリも消さずに両方残す**。
7. **公開を確認する** — GitHub Pages のビルド完了まで待ち、公開URLに新しい `id` と
   画像（HTTP 200）が出ることを確認してから完了を報告する。

   ```bash
   until curl -sS "https://kurolab1028-g.github.io/public_website/favorite-manga.html?cb=$RANDOM" \
     | grep -q "<新しいid>"; do sleep 10; done; echo PUBLISHED
   ```

作品の削除・評価やコメントの変更など、追加以外の依頼でも同じく公開まで行う。
