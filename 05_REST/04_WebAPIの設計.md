# 第5章-04：Web API の設計

---

## 1. Web API とは

Web API（Application Programming Interface）は、
Web を通じて機能・データを外部プログラムに提供するインターフェースです。

```
ブラウザ  ┐
iOS アプリ  ├ → Web API → サーバーのデータ・機能
Android アプリ ┘
外部サービス
```

Web API があることで、同じバックエンドに対して様々なクライアントがアクセスできます。

---

## 2. REST API vs GraphQL vs gRPC

現在の主要な API 設計アプローチを比較します。

### REST API

HTTP + JSON を使う最もポピュラーなスタイルです。

```
GET /users/42
← {"id": 42, "name": "Kazuki", "email": "..."}
```

メリット：学習コストが低い・キャッシュが容易・ブラウザから直接使える
デメリット：オーバーフェッチ・アンダーフェッチが起きやすい

### GraphQL

Facebook が開発した、クライアントが必要なフィールドを指定できるクエリ言語です。

```graphql
query {
  user(id: 42) {
    name
    orders {
      id
      total
    }
  }
}
```

メリット：必要なデータだけ取得できる・型システムが強力
デメリット：学習コストが高い・キャッシュが複雑

### gRPC

Google が開発した、Protocol Buffers を使う高性能な RPC フレームワークです。

```proto
rpc GetUser (UserRequest) returns (UserResponse);
```

メリット：高速・型安全・ストリーミング対応
デメリット：ブラウザから直接使いにくい・学習コストが高い

---

## 3. REST API の設計プロセス

### Step 1：リソースの洗い出し

システムが扱うエンティティ（名詞）を列挙します。

```
EC サイトの場合:
- 商品（Product）
- ユーザー（User）
- 注文（Order）
- 注文明細（OrderItem）
- カート（Cart）
- カートアイテム（CartItem）
- レビュー（Review）
- カテゴリー（Category）
```

### Step 2：リソース間の関係を整理

```
User ─── has many ─── Order
Order ─── has many ─── OrderItem
OrderItem ─── belongs to ─── Product
Product ─── belongs to ─── Category
Product ─── has many ─── Review（User が書く）
```

### Step 3：URI の設計

```
/users                    ユーザー一覧・作成
/users/:id                特定ユーザー

/products                 商品一覧・作成
/products/:id             特定商品
/products/:id/reviews     商品のレビュー一覧・作成

/orders                   注文一覧・作成
/orders/:id               特定注文
/orders/:id/items         注文明細

/cart                     現在のカート（ユーザーに紐づく）
/cart/items               カートアイテム一覧・追加
/cart/items/:id           カートアイテム更新・削除
```

### Step 4：各エンドポイントのメソッドと振る舞いを定義

---

## 4. API ドキュメント

API を公開するときはドキュメントが不可欠です。

### OpenAPI（旧 Swagger）

API の仕様を YAML または JSON で記述する標準仕様です。

```yaml
openapi: 3.0.0
info:
  title: Example API
  version: 1.0.0

paths:
  /users/{id}:
    get:
      summary: ユーザーを取得する
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: integer
      responses:
        '200':
          description: 成功
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '404':
          description: ユーザーが見つからない

components:
  schemas:
    User:
      type: object
      properties:
        id:
          type: integer
        name:
          type: string
        email:
          type: string
```

OpenAPI 仕様から Swagger UI などのインタラクティブなドキュメントを自動生成できます。

---

## 5. API の認証設計

### 認証方式の選択

| 用途 | 推奨方式 |
|------|---------|
| ユーザー向けアプリ（ブラウザ）| Cookie + Session または JWT |
| モバイルアプリ | JWT（Bearer Token）|
| サービス間連携 | API キー または OAuth 2.0 クライアント認証 |
| 外部開発者向け API | OAuth 2.0 または API キー |

### API キーの管理

```http
GET /data HTTP/1.1
X-API-Key: sk_live_abc123def456...
```

API キーの命名規則の例（Stripe 等が採用）：

```
sk_live_... → 本番用シークレットキー（サーバーサイドのみ）
sk_test_... → テスト用シークレットキー
pk_live_... → 本番用公開キー（クライアントサイドで使用可）
```

---

## 6. ページネーション

大量データを分割して返す設計パターンです。

### オフセットページネーション

```
GET /articles?page=2&per_page=20

レスポンス:
{
  "data": [...],
  "pagination": {
    "page": 2,
    "per_page": 20,
    "total": 100,
    "total_pages": 5
  }
}
```

シンプルですが、データ追加・削除があると結果がずれる問題があります。

### カーソルページネーション

```
GET /articles?cursor=eyJpZCI6MjB9&limit=20

レスポンス:
{
  "data": [...],
  "next_cursor": "eyJpZCI6NDB9",
  "has_more": true
}
```

無限スクロールや大量データに適しています。

---

## 7. フィルタリング・ソート

```
# 複数条件でフィルタリング
GET /articles?status=published&author_id=42&tag=web

# ソート
GET /articles?sort=created_at&order=desc

# 複数フィールドでのソート
GET /articles?sort=views:desc,created_at:asc

# 特定フィールドのみ取得（スパースフィールドセット）
GET /users/42?fields=id,name,email
```

---

## 8. レート制限（Rate Limiting）

API の乱用防止のために、リクエスト数を制限します。

```http
HTTP/1.1 200 OK
X-RateLimit-Limit: 1000        ← 1時間あたりの上限
X-RateLimit-Remaining: 995     ← 残りリクエスト数
X-RateLimit-Reset: 1745600000  ← リセット時刻（Unix タイムスタンプ）

（制限超過時）
HTTP/1.1 429 Too Many Requests
Retry-After: 3600
```

---

## 9. エラーハンドリングの設計

一貫したエラーレスポンスの形式が重要です。

```json
{
  "error": {
    "status": 422,
    "code": "VALIDATION_ERROR",
    "message": "入力値が正しくありません",
    "details": [
      {
        "field": "email",
        "code": "INVALID_FORMAT",
        "message": "メールアドレスの形式が正しくありません"
      }
    ],
    "request_id": "req_abc123"    ← デバッグ用のリクエスト ID
  }
}
```

### エラーコードの設計

HTTP ステータスコードだけでなく、アプリ固有のエラーコードを持つと便利です。

```
USER_NOT_FOUND
EMAIL_ALREADY_EXISTS
INSUFFICIENT_BALANCE
PAYMENT_FAILED
SESSION_EXPIRED
```

クライアントはこのコードを見て適切なメッセージを表示できます。

---

## 10. API 設計のベストプラクティスまとめ

```
URI 設計:
○ /users/42               （リソース + ID）
○ /users/42/orders        （階層関係）
○ /users?sort=name        （一覧 + フィルタリング）
× /getUser?id=42          （動詞を含む）
× /user_list.php          （実装詳細を含む）

HTTP メソッド:
○ 取得: GET / 作成: POST / 更新: PUT・PATCH / 削除: DELETE
× 何でも POST

ステータスコード:
○ 作成成功: 201 Created（Location ヘッダー付き）
○ 削除成功: 204 No Content
○ バリデーションエラー: 422 Unprocessable Entity
× 何でも 200 OK（エラーでも 200 を返す）

レスポンス:
○ 一貫した JSON 形式
○ エラー時も構造化されたエラーレスポンス
× エラーを HTML で返す

認証:
○ Authorization: Bearer {token}
○ Cookie（ブラウザアプリの場合）
× URL パラメーターにトークン・パスワードを含める
```

---

## まとめ

- Web API はプログラムからアクセスするインターフェース
- REST / GraphQL / gRPC の3つが主要なスタイル
- 設計プロセス：リソース定義 → 関係整理 → URI 設計 → メソッド定義
- OpenAPI で API 仕様をドキュメント化する
- ページネーション・フィルタリング・ソート・レート制限の設計が重要
- エラーレスポンスは一貫した形式にする
