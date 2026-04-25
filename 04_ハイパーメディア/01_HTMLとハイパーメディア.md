# 第4章-01：HTML とハイパーメディア

---

## 1. ハイパーメディアとは

**ハイパーテキスト** は、文書同士をリンクで結んだテキストです。
**ハイパーメディア** は、テキストだけでなく画像・音声・動画もリンクで結んだメディアです。

Web の革命性は、文書同士が URI というリンクで結びついている点にあります。
これが「World Wide Web（世界的な蜘蛛の巣）」という名前の由来です。

---

## 2. HTML とは

HTML（HyperText Markup Language）は、Web ページの構造と内容を記述するための言語です。

「マークアップ（Markup）」とは、文書の各部分に意味・構造を付けることです。

```html
<h1>これは見出し</h1>
<p>これは段落です。<strong>ここを強調</strong>しています。</p>
<a href="https://example.com/">リンク</a>
```

---

## 3. HTML の歴史

| バージョン | 年 | 主な特徴 |
|---------|----|---------| 
| HTML 1.0 | 1993 | 最初の HTML |
| HTML 2.0 | 1995 | フォーム・画像のサポート |
| HTML 3.2 | 1997 | テーブル・スクリプト |
| HTML 4.01 | 1999 | CSS との役割分担が明確化 |
| XHTML 1.0 | 2000 | XML ベースの HTML |
| HTML5 | 2014 | セマンティック要素・ビデオ・Canvas・API |
| HTML Living Standard | 現在 | WHATWG が継続的にメンテナンス |

---

## 4. HTML の基本構造

```html
<!DOCTYPE html>               ← 文書型宣言（HTML5 の場合は必ずこの形）
<html lang="ja">              ← ルート要素。言語を指定
  <head>                      ← メタ情報（ブラウザや検索エンジン向け）
    <meta charset="UTF-8">    ← 文字エンコーディング
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ページタイトル</title>
    <link rel="stylesheet" href="style.css">
  </head>
  <body>                      ← 表示されるコンテンツ
    <h1>見出し</h1>
    <p>段落テキスト</p>
  </body>
</html>
```

---

## 5. HTML の主要な要素

### 見出し（h1〜h6）

文書の構造を表します。h1 が最も重要な見出しで、h6 が最も小さい見出し。

```html
<h1>大見出し</h1>
<h2>中見出し</h2>
<h3>小見出し</h3>
```

SEO 上、h1 はページに1つだけにするのが推奨されます。

### 段落（p）

テキストのまとまりを表します。

```html
<p>これは段落です。文章のまとまりを表します。</p>
```

### リンク（a）

ハイパーリンクを作ります。Web の根幹をなす要素です。

```html
<a href="https://example.com/">外部サイトへ</a>
<a href="/about">同じサイトの別ページ</a>
<a href="#section2">同じページの別の位置へ</a>
<a href="mailto:user@example.com">メールを送る</a>
```

`href` 属性に URI を指定します。

### 画像（img）

画像を表示します。

```html
<img src="photo.jpg" alt="山の写真" width="800" height="600">
```

`alt` 属性は代替テキストで、画像が表示できない場合や
スクリーンリーダー（視覚障害者向け）のために必須です。

### リスト（ul・ol・li）

```html
<!-- 順不同リスト -->
<ul>
  <li>りんご</li>
  <li>みかん</li>
</ul>

<!-- 順序付きリスト -->
<ol>
  <li>まず水を沸騰させる</li>
  <li>麺を入れる</li>
</ol>
```

### テーブル（table）

表形式のデータを表示します。

```html
<table>
  <thead>
    <tr>
      <th>名前</th>
      <th>年齢</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>松尾 一輝</td>
      <td>21</td>
    </tr>
  </tbody>
</table>
```

### フォーム（form）

ユーザーからのデータ入力を受け付けます。

```html
<form action="/users" method="post">
  <label for="name">名前:</label>
  <input type="text" id="name" name="name" required>
  
  <label for="email">メール:</label>
  <input type="email" id="email" name="email">
  
  <button type="submit">送信</button>
</form>
```

`action` は送信先の URI、`method` は HTTP メソッド（get または post）を指定します。

---

## 6. HTML5 のセマンティック要素

HTML5 では、文書の構造を意味的に表すセマンティック要素が追加されました。

```html
<header>         ← ページや記事のヘッダー
<nav>            ← ナビゲーションリンク
<main>           ← メインコンテンツ（ページに1つ）
<article>        ← 独立したコンテンツ（記事・ブログ投稿等）
<section>        ← テーマごとのセクション
<aside>          ← サイドバーなどの補足コンテンツ
<footer>         ← ページや記事のフッター
```

従来の `<div>` だけでは構造がわかりませんでしたが、
セマンティック要素を使うと文書の構造が明確になります。

```html
<!-- セマンティックな HTML 構造の例 -->
<body>
  <header>
    <nav>
      <a href="/">ホーム</a>
      <a href="/about">概要</a>
    </nav>
  </header>
  
  <main>
    <article>
      <h1>記事タイトル</h1>
      <p>記事本文...</p>
    </article>
    
    <aside>
      <h2>関連記事</h2>
    </aside>
  </main>
  
  <footer>
    <p>© 2026 Example Inc.</p>
  </footer>
</body>
```

---

## 7. HTML とメディアタイプ

HTML のメディアタイプは `text/html` です。

HTTP レスポンスで次のように指定します。

```http
Content-Type: text/html; charset=UTF-8
```

`charset=UTF-8` を省略すると、ブラウザが文字エンコーディングを推測して
文字化けが発生することがあるため、必ず指定します。

---

## 8. HTML の重要性：リンクによる情報の結合

Web の最大の特徴は、**リンクを通じて世界中の情報が結びついている** ことです。

```html
<a href="https://wikipedia.org/wiki/HTTP">HTTP について</a>
```

このような一行が、世界中の無数のページを結びつけています。

REST（RESTful な設計）では、この「リンクによるリソースの接続」が非常に重要です。
HATEOAS（Hypermedia As The Engine Of Application State）という概念で、
API レスポンスにも次のアクションへのリンクを含めることが推奨されます。

```json
{
  "id": 42,
  "name": "Kazuki Matsuo",
  "_links": {
    "self": {"href": "/users/42"},
    "orders": {"href": "/users/42/orders"},
    "edit": {"href": "/users/42/edit"}
  }
}
```

---

## まとめ

- HTML は Web ページの構造と内容を記述するマークアップ言語
- `<a href="">` によるリンクが Web の「ハイパーテキスト」の本質
- HTML5 のセマンティック要素（header・main・article・footer 等）で文書の意味が明確になる
- メディアタイプは `text/html; charset=UTF-8`
- フォームで GET・POST リクエストを送ることができる
