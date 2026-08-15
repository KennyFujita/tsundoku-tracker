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
- **分類の内訳** — 「読んだ本」と「これから読む本」それぞれのジャンル構成をドーナツで表示。
  分類は国立国会図書館の **NDC（日本十進分類法）** をそのまま使う（技術・自然科学・文学…の10分類）

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

ISBN を調べたときだけ、以下に問い合わせる。それ以外の通信は一切ない。API キーはどれも要らない。

3つを順に引いて、欠けている項目だけを次の取得先で埋めていく。**得意分野が違うので3つとも要る。**

| 問い合わせ先 | 役割 |
|---|---|
| [openBD](https://openbd.jp/) | **新刊**。発売前から登録されるので、出たばかりの本はここでしか拾えない |
| [国立国会図書館サーチ](https://ndlsearch.ndl.go.jp/) | **著者とページ数**。納本制度で収録が厚い。ただし新刊は目録作成のラグで載っていない |
| [Google Books API](https://developers.google.com/books) | **洋書**の受け皿 |

画面には「**〜から取得**」と、どこが答えたかを出す。取れなかった項目も名指しで知らせるので、
うまくいかないときに原因が分かる。

### 取得できる項目の当たり外れ

- **書名** — だいたい取れる
- **著者** — 国立国会図書館なら自然な表記（`夏目漱石 作`）で取れる。openBD は図書館の典拠形
  （`三浦,宏文,1938-2020`）で返ってくるので、読める形に整えている
- **総ページ数** — 国立国会図書館の `dcterms:extent` が唯一まともな取得先。openBD はほぼ持っておらず
  （12冊試して1冊）、Google Books も日本語書籍では欠けていることが多い

- **分類（NDC）** — 国立国会図書館だけが持っている。手元で6冊試して5冊取れた。
  openBD の Cコードは試した範囲では常に空だった

**新刊は国立国会図書館にまだ載っていない**ため、ページ数や分類が取れないことがある。
そのときは本の最後をめくって手で入れる。どの取得先も応答しなくても、エラーで止まらず手入力に倒れる。

### 分類を後から取り込む

先に登録した本には分類が付いていない。「分類の内訳」の下に出る
**「国立国会図書館から取り込む」**を押すと、ISBN のある本をまとめて調べ直す。
それでも付かない本（ISBN 無し・国会図書館に未収録）は、その場のメニューから手で選べる。

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
