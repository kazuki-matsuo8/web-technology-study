# 第3章-04：HTTP ヘッダー

---

## 1. HTTP ヘッダーとは

HTTP ヘッダーは、リクエストやレスポンスに **付加的な情報（メタデータ）** を付けるための仕組みです。

ボディがリソースの「内容」を運ぶのに対し、
ヘッダーはその内容に関する「説明」を運びます。

```http
HTTP/1.1 200 OK
Content-Type: application/json     ← ボディの形式
Content-Length: 52                 ← ボディのバイト数
Cache-Control: max-age=3600        ← キャッシュ設定
Date: Mon, 21 Apr 2026 10:00:00 GMT ← 日時

{"id": 42, "name": "Kazuki"}      ← ボディ（内容）
```

ヘッダーは `フィールド名: 値` の形式で記述します（大文字小文字は区別しない）。

---

## 2. ヘッダーの種類

| 種類 | 説明 |
|------|------|
| 一般ヘッダー | リクエスト・レスポンス両方で使える |
| リクエストヘッダー | クライアントがサーバーに送る情報 |
| レスポンスヘッダー | サーバーがクライアントに送る情報 |
| エンティティヘッダー | ボディに関する情報（Content-Type など）|

---

## 3. コンテンツ関連ヘッダー

### Content-Type

ボディのメディアタイプ（形式）を示します。
MIME タイプ（Multipurpose Internet Mail Extensions）の形式で記述します。

```http
Content-Type: text/html; charset=UTF-8
Content-Type: application/json
Content-Type: application/xml
Content-Type: image/jpeg
Content-Type: video/mp4
Content-Type: multipart/form-data; boundary=----FormBoundary7MA4YWxkTrZu0gW
```

`charset=UTF-8` は文字エンコーディングの指定です。
HTML では必ず指定することが重要です（文字化けを防ぐ）。

よく使う MIME タイプ：

| MIME タイプ | 説明 |
|-----------|------|
| `text/html` | HTML |
| `text/plain` | プレーンテキスト |
| `text/css` | CSS |
| `application/javascript` | JavaScript |
| `application/json` | JSON |
| `application/xml` | XML |
| `image/jpeg` | JPEG 画像 |
| `image/png` | PNG 画像 |
| `image/svg+xml` | SVG 画像 |
| `application/pdf` | PDF |
| `multipart/form-data` | ファイルアップロード付きフォーム |

### Content-Length

ボディのバイト数を示します。

```http
Content-Length: 1024
```

HTTP/1.1 ではボディがある場合に必須です。
ただし、チャンク転送エンコーディングを使う場合は省略できます。

### Content-Encoding

ボディの **圧縮形式** を示します。

```http
Content-Encoding: gzip
Content-Encoding: br       （Brotli、より圧縮率が高い）
```

サーバーがボディを圧縮して送ることで、転送量を削減できます。

### Content-Language

ボディの言語を示します。

```http
Content-Language: ja
Content-Language: en-US
```

---

## 4. リクエスト関連ヘッダー

### Host

**必須ヘッダー** です（HTTP/1.1 から）。
アクセス先のホスト名を示します。

```http
Host: api.example.com
Host: api.example.com:8080   （ポートが 80/443 以外の場合）
```

1つのサーバーで複数のドメインを運用（バーチャルホスト）する際、
Host ヘッダーでどのドメインへのリクエストかを判別します。

### Accept

クライアントが **受け入れ可能なメディアタイプ** を示します。

```http
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept: application/json
```

`q=` は品質係数（Quality Factor）で、優先度を示します（0〜1、高いほど優先）。

### Accept-Language

受け入れ可能な言語を示します。

```http
Accept-Language: ja,en-US;q=0.9,en;q=0.8
```

### Accept-Encoding

受け入れ可能な圧縮形式を示します。

```http
Accept-Encoding: gzip, deflate, br
```

### User-Agent

クライアント（ブラウザやアプリ）の情報を示します。

```http
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36
User-Agent: curl/8.1.0
User-Agent: MyApp/1.0.0 (iOS 17.0)
```

サーバーはこれを見てブラウザの種類を判別できますが、
ユーザーエージェントスニッフィング（UA を見て処理を分ける）は
現在は避けるべきとされています（代わりにコンテントネゴシエーションを使う）。

### Referer（リファラー）

リクエスト前にいたページの URI を示します（「どこから来たか」）。

```http
Referer: https://google.com/search?q=web技術
```

プライバシー上の理由から、HTTPS から HTTP へのリクエストでは省略されます。
（Referer という名前のスペルミスは RFC を策定した時点で気づいたが、既に普及していたため修正されなかった）

### Authorization

認証情報を送ります。詳しくは第3章-05 認証で扱います。

```http
Authorization: Bearer eyJhbGciOiJSUzI1NiJ9...
Authorization: Basic dXNlcjpwYXNzd29yZA==
```

---

## 5. レスポンス関連ヘッダー

### Location

リダイレクト先の URI や、作成したリソースの URI を示します。

```http
HTTP/1.1 201 Created
Location: /users/42

HTTP/1.1 301 Moved Permanently
Location: https://example.com/new-path
```

### Server

サーバーソフトウェアの情報を示します。

```http
Server: nginx/1.24.0
Server: Apache/2.4.58
```

セキュリティのため、本番環境では詳細なバージョン情報を隠すことが推奨されます。

### WWW-Authenticate

認証が必要であることをクライアントに伝えます（401 と一緒に返す）。

```http
HTTP/1.1 401 Unauthorized
WWW-Authenticate: Bearer realm="api.example.com"
```

### Allow

そのリソースで使用可能なメソッドを示します（405 と一緒に返す）。

```http
HTTP/1.1 405 Method Not Allowed
Allow: GET, HEAD, PUT
```

---

## 6. キャッシュ関連ヘッダー

詳細は第3章-06 キャッシュで扱いますが、主要なものを紹介します。

### Cache-Control

キャッシュの振る舞いを制御します。

```http
Cache-Control: max-age=3600         （3600秒キャッシュ可）
Cache-Control: no-cache             （キャッシュしてよいが毎回検証）
Cache-Control: no-store             （キャッシュしてはいけない）
Cache-Control: private              （個人のキャッシュのみ可）
Cache-Control: public               （CDN 等のキャッシュも可）
```

### ETag

リソースの **バージョン識別子** です。
リソースが変更されると ETag も変わります。

```http
ETag: "abc123def456"
ETag: W/"abc123"   （Weak ETag: バイト単位の一致でなく意味的な一致）
```

### Last-Modified

リソースの **最終更新日時** です。

```http
Last-Modified: Mon, 20 Apr 2026 12:00:00 GMT
```

---

## 7. CORS 関連ヘッダー

CORS（Cross-Origin Resource Sharing）は、
異なるオリジン間でのリクエストを制御する仕組みです。

```http
（レスポンスに含めることでクロスオリジンを許可）

Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Content-Type, Authorization
Access-Control-Max-Age: 3600
```

---

## 8. セキュリティ関連ヘッダー

### Strict-Transport-Security（HSTS）

このサイトには **HTTPS のみでアクセスすべき** ことをブラウザに伝えます。

```http
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

### X-Content-Type-Options

ブラウザが **MIME タイプのスニッフィングを行わない** ようにします。

```http
X-Content-Type-Options: nosniff
```

### X-Frame-Options

**クリックジャッキング攻撃** を防ぐため、iframe での埋め込みを制限します。

```http
X-Frame-Options: DENY              （iframe 埋め込み禁止）
X-Frame-Options: SAMEORIGIN        （同一オリジンのみ許可）
```

### Content-Security-Policy（CSP）

読み込めるリソースの **ソースを制限** します。XSS 攻撃の緩和に重要です。

```http
Content-Security-Policy: default-src 'self'; script-src 'self' cdn.example.com
```

---

## まとめ

| カテゴリー | 主要ヘッダー |
|-----------|-------------|
| コンテンツ | Content-Type / Content-Length / Content-Encoding |
| リクエスト | Host / Accept / Accept-Language / Authorization |
| レスポンス | Location / WWW-Authenticate / Allow |
| キャッシュ | Cache-Control / ETag / Last-Modified |
| セキュリティ | Strict-Transport-Security / X-Frame-Options / CSP |

ヘッダーはリクエスト・レスポンスに追加情報を付加するもの。
コンテンツタイプ・認証・キャッシュ・セキュリティなど様々な用途がある。
