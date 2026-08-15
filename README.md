# 積読と進捗

読みかけの本が何冊あって、あとどれだけ残っているかを把握するための道具。
HTML ファイル1枚で動く。ビルド工程も外部ライブラリもサーバーも要らない。

<https://kennyfujita.github.io/tools/tsundoku/>

## できること

- **積読の山** — 未読の本を「厚み＝ページ数」の背表紙として積み上げて表示する。残量が一目で分かる
- **進捗の記録** — 「読んだところまで」のページ番号を入れるだけ。同じ日に複数回入れたら上書きされる
- **ペースの計算** — 直近14日の実績から 1日あたり何ページ読んでいるかを出し、この調子だといつ読み終わるかを示す
- **目標日との比較** — 読了目標日を入れると「必要 p/日」と実績を比べて **順調 / 遅れ** を表示する
- **ISBN から登録** — 本の裏の13桁を入れると書名と著者が埋まる。Chrome ならカメラでバーコードを読める。
  **総ページ数は手で入れる**（後述）
- **集計** — 積読冊数 / 未読総ページ / 読書中冊数 / 直近7日に読んだページ / 今年の読了冊数

## 使い方

1. ページを開く
2. 「本を追加する」→ ISBN を入れて「調べる」（または手で書名・総ページを入力）→「積む」
3. 読んだら、その本の「読んだところまで」に到達ページを入れて「記録」
4. 総ページまで行くと自動で「読了」になる

状態は **積読 / 読書中 / 読了 / 中断** の4つ。初めてページを記録すると積読から読書中に変わる。

### カメラでバーコードを読む

`BarcodeDetector` API を使っている。**Chrome（Android / デスクトップ）で使える。**
Safari は未対応なので、その場合はボタン自体が出ず、ISBN の手入力になる。

カメラは HTTPS か `localhost` でしか使えない。ファイルをダブルクリックして `file://` で開いた場合も
ボタンは出ない（これは正しい挙動）。

## データはどこにあるか

**入力した内容はブラウザの localStorage（キー `tsundoku.v1`）にだけ保存される。**
サーバーには何も送っていない。作者も他人も中身を見ることはできない。

ISBN を調べたときだけ、以下に問い合わせる。それ以外の通信は一切ない。どちらも API キーは要らない。

| 問い合わせ先 | 取得するもの | 実際の当たり具合 |
|---|---|---|
| [openBD](https://openbd.jp/) | 書名・著者 | 手元で12冊試して **11冊** で書名が取れた |
| [Google Books API](https://developers.google.com/books) | ページ数の補完 | 期待しない方がよい（下記） |

### 総ページ数だけは手で入れる

書誌データベースはページ数をあまり持っていない。openBD の `Extent` は12冊試して**1冊しか入っていなかった**。
Google Books はキー無しだと共有クォータですぐ 429 になり、日本語書籍の `pageCount` も欠けていることが多い。

**総ページ数は、いま手に持っている本の最後をめくれば分かる。** そこだけ手で入れる仕様にしている。
自動化して本当に効くのは「長い和書のタイトルと著者名を打たなくて済む」ことで、そこは openBD で足りている。

Google Books が応答しなくても、エラーで止まらず手入力に倒れる。

### 控えを取る・持ち運ぶ

localStorage は**ブラウザの閲覧データを削除すると消える**。
定期的に「**書き出す（JSON）**」でファイルに保存しておくこと。

別の端末やブラウザに移すときも、書き出した JSON を移動先で「**読み込む**」で復元できる。
読み込むと既存のデータは置き換わる（実行前に確認が出る）。

## 手元に置いて使う

このページは1枚で完結しているので、保存すればそのまま自分の道具にできる。

1. ページを右クリック →「別名で保存」（または [`index.html`](index.html) を Raw で保存）
2. 保存したファイルをダブルクリックで開く

`file://` で開くとカメラだけ使えないが、それ以外はすべて動く。
自分で書き換えて使ってもらってかまわない。

## 作りについて

- HTML + CSS + JavaScript を1ファイルに内包。依存ライブラリなし、ビルド工程なし
- 色はすべて CSS 変数。`prefers-color-scheme` と `:root[data-theme]` の両方に対応しているので、
  ライトでもダークでも読める
- 書体は OS 標準のもののみ（Web フォントを読み込まない）。書名は明朝、UI はゴシック、数値は等幅
- このサイトの `assets/style.css` は読み込んでいない。1枚で完結させるため意図的に独立させている

## ライセンス

MIT License

Copyright (c) 2026 Kento Fujita

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
