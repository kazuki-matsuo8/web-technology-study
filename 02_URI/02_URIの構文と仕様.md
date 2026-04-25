# 第2章-02：URI の構文と仕様

---

## 1. URI の基本構文

URI は RFC 3986 で定義されています。基本的な構文は次のとおりです。

```
scheme ":" hier-part [ "?" query ] [ "#" fragment ]

例:
https://user:pass@example.com:8080/users/42?sort=name#section1
│      │         │           │    │        │          │
scheme  userinfo  host        port path     query      fragment
```

---

## 2. 各コンポーネントの詳細

### スキーム（Scheme）

URI の種類・プロトコルを示します。`://` の前の部分です。

| スキーム | 用途 |
|---------|------|
| `https` | HTTPS（暗号化された HTTP）|
| `http` | HTTP |
| `mailto` | 電子メール |
| `ftp` | FTP ファイル転送 |
| `tel` | 電話番号 |
| `file` | ローカルファイル |

```
https://example.com/
mailto:user@example.com
tel:+81-90-1234-5678
file:///home/user/document.txt
```

スキームは大文字小文字を区別しません（`HTTPS` も `https` も同じ）。

### ユーザー情報（Userinfo）

`ホスト名` の前に `user:password@` の形式で認証情報を含められます。

```
https://admin:secret@example.com/
```

ただし、パスワードを URI に含めることはセキュリティ上の問題があるため
現在はほとんど使われません。

### ホスト（Host）

アクセス先のサーバーを示します。IP アドレスまたはドメイン名で指定します。

```
example.com          （ドメイン名）
192.168.1.1          （IPv4 アドレス）
[2001:db8::1]        （IPv6 アドレス。角括弧で囲む）
```

### ポート番号（Port）

接続先のポートを指定します。省略時はスキームのデフォルトポートを使います。

| スキーム | デフォルトポート |
|---------|----------------|
| `http` | 80 |
| `https` | 443 |
| `ftp` | 21 |

```
https://example.com:8080/   → ポート 8080 に接続
https://example.com/        → ポート 443 に接続（省略）
```

### パス（Path）

サーバー内でのリソースの場所を示します。`/` で区切って階層を表現します。

```
/                      → ルート
/users                 → ユーザー一覧
/users/42              → ID が42のユーザー
/users/42/orders       → ID が42のユーザーの注文一覧
```

パスは必ず `/` で始まります。

### クエリ（Query）

`?` に続く部分で、追加的なパラメーターを渡します。

```
/search?q=web&sort=date&page=2
```

`key=value` の形式で記述し、複数のパラメーターは `&` でつなぎます。

用途：
- 検索条件（`?q=web`）
- ソート順（`?sort=date`）
- ページネーション（`?page=2`）
- フィルタリング（`?status=active`）

### フラグメント（Fragment）

`#` に続く部分で、リソース内の特定の位置を示します。

```
https://example.com/docs/api#authentication
```

重要な特徴として、**フラグメントはサーバーに送信されない** ことがあります。
ブラウザがローカルで処理し、ページ内の該当位置にスクロールします。

---

## 3. URI のエンコーディング

URI に使用できる文字には制限があります。
制限外の文字（日本語など）は **パーセントエンコーディング** で表現します。

```
パーセントエンコーディングの形式: %XX
（XX は文字の UTF-8 バイトの16進数表記）

例：「東京」→ %E6%9D%B1%E4%BA%AC
```

### URI に使える文字（予約なし文字）

```
A-Z  a-z  0-9  -  _  .  ~
```

これらは常にそのまま使えます。

### 予約文字（特別な意味を持つ文字）

```
:  /  ?  #  [  ]  @  !  $  &  '  (  )  *  +  ,  ;  =
```

これらはURI の構造を分けるために使われます。
パス・クエリの「値」としてこれらを使いたい場合はエンコードが必要です。

### エンコードの例

```
/search?q=Web入門
    ↓ エンコード
/search?q=Web%E5%85%A5%E9%96%80

/users/田中 太郎
    ↓ エンコード
/users/%E7%94%B0%E4%B8%AD%20%E5%A4%AA%E9%83%8E
（スペースは %20 または + でエンコード）
```

---

## 4. 絶対 URI と相対 URI

### 絶対 URI

スキームから始まる完全な URI です。

```
https://example.com/users/42
```

### 相対 URI（相対参照）

現在の URI を基準にして表現した URI です。HTML などで使われます。

```html
<!-- 現在のページが https://example.com/users/ の場合 -->

<a href="42">          → https://example.com/users/42
<a href="../products"> → https://example.com/products
<a href="/about">      → https://example.com/about
<a href="//cdn.example.com/img.jpg"> → https://cdn.example.com/img.jpg
```

相対 URI は HTMLなど文書内でのリンクに使いやすいですが、
API では絶対 URI を使うのが一般的です。

---

## 5. URI の正規化

同一のリソースを示す URI でも、表現が異なることがあります。
**正規化（Normalization）** により、同一の URI に統一します。

```
http://Example.COM/Users/42    （ホストの大文字小文字）
http://example.com/Users/42    ← 正規化: ホストは小文字
http://example.com/users/42    ← 正規化: パスの大文字小文字

http://example.com:80/users/42  （デフォルトポートの明示）
http://example.com/users/42    ← 正規化: デフォルトポートは省略

https://example.com/users/%4b  （エンコードされた A-Z a-z 0-9 - _ . ~）
https://example.com/users/K    ← 正規化: 不要なエンコードを解除
```

---

## 6. URI とセキュリティ

### オープンリダイレクト

URI をパラメーターとして受け取り、そこへリダイレクトするときは注意が必要です。

```
https://example.com/login?next=https://evil.com/phishing
```

`next` パラメーターを検証せずにリダイレクトすると、
悪意あるサイトへ誘導される脆弱性（オープンリダイレクト）になります。

### URI インジェクション

ユーザー入力を URI に組み込む際、不正な文字を含む入力を
適切にエンコードしないと予期しない URI が生成される危険があります。

---

## まとめ

URI の構成要素：
```
スキーム :// ユーザー情報 @ ホスト : ポート / パス ? クエリ # フラグメント
```

- スキーム：プロトコルの種類（http, https など）
- ホスト：アクセス先のサーバー
- パス：リソースの場所
- クエリ：追加パラメーター（`?key=value`）
- フラグメント：リソース内の位置（`#section`、サーバーには送られない）
- 日本語などはパーセントエンコーディングが必要
