# 第5章-03：RESTful な設計

---

## 1. RESTful な設計の目標

RESTful な設計とは、REST の原則に従って Web サービスを設計することです。

目標は：
- URI でリソースを適切に表現する
- HTTP メソッドを正しく使う
- HTTP ステータスコードを正しく使う
- 適切なヘッダーを使う

---

## 2. リソース設計

### リソースの識別

まず「何がリソースか」を定義します。

```
システム：ブログ

リソース一覧：
- 記事（Article）
- ユーザー（User）
- コメント（Comment）
- タグ（Tag）
- カテゴリー（Category）
```

### URI の設計

リソースごとに URI を設計します。

```
記事:
  GET    /articles        → 記事一覧
  POST   /articles        → 記事作成
  GET    /articles/:id    → 特定の記事
  PUT    /articles/:id    → 記事の全更新
  PATCH  /articles/:id    → 記事の部分更新
  DELETE /articles/:id    → 記事の削除

コメント（記事に紐づく子リソース）:
  GET    /articles/:id/comments     → ある記事のコメント一覧
  POST   /articles/:id/comments     → コメント追加
  DELETE /articles/:id/comments/:cid → コメント削除
```

### リソース名は複数形で

コレクション（一覧）を表す URI は複数形にするのが一般的です。

```
○ /users      （複数形）
× /user       （単数形は非推奨）

○ /articles/42/comments
× /article/42/comment
```

---

## 3. HTTP メソッドとリソース操作の対応

| 操作 | メソッド | URI 例 |
|------|---------|-------|
| 一覧取得 | GET | /articles |
| 作成 | POST | /articles |
| 取得 | GET | /articles/42 |
| 全更新 | PUT | /articles/42 |
| 部分更新 | PATCH | /articles/42 |
| 削除 | DELETE | /articles/42 |

---

## 4. ステータスコードの正しい使い方

```
GET    /articles/42   → 200 OK（記事を返す）
POST   /articles      → 201 Created（作成成功。Location ヘッダーで URI を返す）
PUT    /articles/42   → 200 OK または 204 No Content
PATCH  /articles/42   → 200 OK または 204 No Content
DELETE /articles/42   → 204 No Content

存在しないリソース:
GET    /articles/999  → 404 Not Found

バリデーションエラー:
POST   /articles      → 422 Unprocessable Entity（エラー詳細を返す）

認証エラー:
GET    /admin/data    → 401 Unauthorized
GET    /others/data   → 403 Forbidden
```

---

## 5. レスポンスの設計

### 一覧レスポンスの設計

```json
GET /articles

{
  "data": [
    {"id": 1, "title": "HTTP について", "created_at": "2026-04-20T10:00:00+09:00"},
    {"id": 2, "title": "REST とは", "created_at": "2026-04-21T10:00:00+09:00"}
  ],
  "pagination": {
    "total": 100,
    "page": 1,
    "per_page": 20,
    "total_pages": 5
  }
}
```

### 個別リソースのレスポンス

```json
GET /articles/42

{
  "id": 42,
  "title": "HTTP について",
  "body": "HTTP とは...",
  "author": {
    "id": 1,
    "name": "松尾 一輝"
  },
  "tags": ["web", "http", "protocol"],
  "created_at": "2026-04-20T10:00:00+09:00",
  "updated_at": "2026-04-26T09:00:00+09:00"
}
```

### エラーレスポンスの設計

エラーレスポンスは一貫した形式にすることが大切です。

```json
HTTP/1.1 422 Unprocessable Entity
Content-Type: application/json

{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "入力値が不正です",
    "details": [
      {"field": "title", "message": "タイトルは必須です"},
      {"field": "body", "message": "本文は10文字以上必要です"}
    ]
  }
}
```

---

## 6. CRUD と HTTP の完全な例

### ユーザーシステムの完全な API 設計

```
# ユーザー一覧取得
GET /users?page=1&per_page=20
→ 200 OK
{
  "data": [{...}, {...}],
  "pagination": {"total": 50, "page": 1, ...}
}

# ユーザー作成
POST /users
Body: {"name": "松尾 一輝", "email": "kazuki@example.com"}
→ 201 Created
Location: /users/42
{
  "id": 42,
  "name": "松尾 一輝",
  "email": "kazuki@example.com",
  "created_at": "2026-04-26T10:00:00+09:00"
}

# ユーザー取得
GET /users/42
→ 200 OK
{
  "id": 42,
  "name": "松尾 一輝",
  ...
}

# ユーザー更新（全置換）
PUT /users/42
Body: {"name": "松尾 一輝（更新後）", "email": "new@example.com"}
→ 200 OK または 204 No Content

# ユーザー部分更新
PATCH /users/42
Body: {"email": "new@example.com"}
→ 200 OK または 204 No Content

# ユーザー削除
DELETE /users/42
→ 204 No Content

# 存在しないユーザー取得
GET /users/999
→ 404 Not Found
{"error": {"message": "ユーザーが見つかりません"}}
```

---

## 7. よくある設計の迷いどころ

### ログインはどう表現するか

ログイン＝「セッションの作成」と考えます。

```
POST /sessions           → ログイン（セッション作成）
DELETE /sessions/current → ログアウト（セッション削除）
```

JWT を使う場合：

```
POST /auth/token    → トークン発行（ログイン）
DELETE /auth/token  → トークン失効（ログアウト）
```

### 「公開する」「承認する」などの操作

リソースの状態変化として表現します。

```
POST /articles/42/publish    → 記事の公開（子リソースとしての操作）

または状態変更として:
PATCH /articles/42
Body: {"status": "published"}
```

後者の方が RESTful です。

### 検索機能

```
GET /articles?q=HTTP&tag=web&author=42&sort=created_at&order=desc
```

クエリパラメーターで検索条件を渡します。
複雑な検索（ボディで条件を渡したい場合）は POST を使うことも。

```
POST /articles/search
Body: {
  "query": "HTTP",
  "tags": ["web", "http"],
  "date_from": "2026-01-01",
  "date_to": "2026-12-31"
}
```

---

## 8. API バージョニング

API を変更する際、後方互換性を保つためにバージョンを管理します。

```
# URI にバージョンを含める（最もよく使われる）
/api/v1/users
/api/v2/users

# ヘッダーでバージョンを指定
Accept: application/vnd.example.v2+json

# クエリパラメーターで指定
/api/users?version=2
```

---

## まとめ

RESTful な設計の原則：

```
1. リソースを名詞で表現する URI を設計する
   /users, /articles, /orders

2. HTTP メソッドで操作を表現する
   GET（取得）POST（作成）PUT/PATCH（更新）DELETE（削除）

3. HTTP ステータスコードを正しく使う
   200 成功 / 201 作成 / 204 コンテンツなし
   400 不正 / 401 未認証 / 403 禁止 / 404 未発見

4. レスポンスを一貫した JSON 形式にする
   成功時: データ / 一覧時: data + pagination / エラー時: error オブジェクト

5. ステートレスに保つ
   認証情報は毎回リクエストに含める
```
