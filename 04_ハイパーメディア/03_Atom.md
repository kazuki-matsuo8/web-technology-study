# 第4章-03：Atom

---

## 1. Atom とは

Atom は Web フィード（ブログや Web サイトの更新情報）を配信するための XML ベースのフォーマットです。
RSS の後継として開発され、RFC 4287 で標準化されています。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<feed xmlns="http://www.w3.org/2005/Atom">
  <title>松尾一輝のブログ</title>
  <link href="https://example.com/blog/"/>
  <updated>2026-04-26T10:00:00+09:00</updated>
  <id>urn:uuid:60a76c80-d399-11d9-b91C-0003939e0af6</id>
  
  <entry>
    <title>HTTP について学んだこと</title>
    <link href="https://example.com/blog/posts/42"/>
    <id>urn:uuid:1225c695-cfb8-4ebb-aaaa-80da344efa6a</id>
    <updated>2026-04-26T09:00:00+09:00</updated>
    <summary>HTTP の基礎を学びました...</summary>
    <author>
      <name>松尾 一輝</name>
      <email>kazuki@example.com</email>
    </author>
  </entry>
</feed>
```

---

## 2. Atom の用途

### Web フィード（ブログ・ニュース配信）

ブログや ニュースサイトが Atom フィードを公開すると、
RSS リーダーやフィードアグリゲーターがそのフィードを購読できます。

```
ブログサーバー → Atom フィード配信
フィードリーダー → フィードを定期的に取得 → 新しい記事を通知
```

### メディアタイプ

```
application/atom+xml
```

---

## 3. Atom フィードの構造

### フィード要素（フィード全体の情報）

| 要素 | 必須 | 説明 |
|------|:----:|------|
| `title` | ○ | フィードのタイトル |
| `id` | ○ | フィードの一意な ID（URI 形式）|
| `updated` | ○ | 最終更新日時（ISO 8601 形式）|
| `link` | △ | フィードの URL |
| `author` | △ | 著者情報 |
| `subtitle` | × | フィードのサブタイトル |

### エントリー要素（個別記事の情報）

| 要素 | 必須 | 説明 |
|------|:----:|------|
| `title` | ○ | 記事タイトル |
| `id` | ○ | 記事の一意な ID |
| `updated` | ○ | 最終更新日時 |
| `link` | × | 記事の URL |
| `summary` | △ | 記事の概要 |
| `content` | × | 記事の本文 |
| `author` | △ | 著者情報 |

---

## 4. Atom と RSS の比較

| 比較 | Atom | RSS 2.0 |
|------|------|---------|
| 標準化 | RFC 4287（IETF）| 独自仕様 |
| 文字エンコーディング | UTF-8 必須 | 指定可能 |
| コンテンツタイプ | HTML・テキスト・XML の明示 | 指定なし |
| 日時フォーマット | ISO 8601 必須 | RFC 2822（あいまい）|
| 一意な ID | `id` 要素（必須）| `guid` 要素（任意）|

Atom の方が仕様が厳格で曖昧さが少なく、実装しやすいとされています。

---

## 5. Atom Publishing Protocol（AtomPub）

Atom の延長として、**Web 上でコンテンツを作成・編集・削除する** ためのプロトコルが
AtomPub（RFC 5023）です。

AtomPub は HTTP + Atom を使って、Web でコンテンツ管理を行います。

```
コレクション（記事一覧）:
GET  /blog/posts         → Atom フィードで一覧取得
POST /blog/posts         → 新規記事作成（Atom エントリーを POST）

メンバー（個別記事）:
GET    /blog/posts/42    → 記事取得（Atom エントリーで返す）
PUT    /blog/posts/42    → 記事更新（Atom エントリーで全置換）
DELETE /blog/posts/42    → 記事削除
```

AtomPub は「Web を書き込み可能にする」という意味で、
REST の設計思想を実践した代表的なプロトコルです。

---

## 6. 現在の Atom の位置づけ

Atom フィードは今も多くのブログやニュースサイトで使われています。
しかし、JSON が普及した現在は、JSON ベースの フィード形式（JSON Feed など）も登場しています。

また、AtomPub は現在はあまり使われず、
各サービス独自の REST API が主流になっています。

---

## まとめ

- Atom は Web フィード（更新情報配信）のための XML フォーマット（RFC 4287）
- ブログ・ニュースサイトが更新情報を配信するために使う
- RSS より仕様が厳格で曖昧さが少ない
- AtomPub は HTTP + Atom で Web コンテンツを CRUD する仕組み（REST の実践例）
- メディアタイプは `application/atom+xml`
