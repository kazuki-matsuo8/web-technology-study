# 第3章-02：HTTP メソッド

---

## 1. HTTP メソッドとは

HTTP メソッドは、クライアントがリソースに対して **何をしたいか** を表します。
URI がリソースの「名前（何に対して）」を表すのに対し、
メソッドは「操作（何をする）」を表します。

```
GET  /users/42
│    │
│    └── リソース：ID が42のユーザー
└── メソッド：取得する
```

---

## 2. 主要な HTTP メソッド一覧

| メソッド | 意味 | リクエストボディ | レスポンスボディ |
|---------|------|----------------|----------------|
| GET | リソースを取得する | なし | あり |
| POST | リソースを作成・追加する | あり | あり |
| PUT | リソースを置き換える（全更新）| あり | あり |
| PATCH | リソースを部分的に更新する | あり | あり |
| DELETE | リソースを削除する | 基本なし | あり |
| HEAD | ヘッダーのみ取得する | なし | なし |
| OPTIONS | 利用可能なメソッドを確認する | なし | あり |

---

## 3. GET

リソースを **取得** するためのメソッドです。
最も基本的かつ頻繁に使われます。

```http
GET /users/42 HTTP/1.1
Host: api.example.com
Accept: application/json
```

### GET の特性

**安全（Safe）**: リソースを変更しません。
何度 GET を送っても、サーバーの状態は変わりません。

**冪等（Idempotent）**: 何度同じリクエストを送っても結果が同じです。

**キャッシュ可能**: GET のレスポンスはキャッシュできます。
ブラウザや CDN が同じ URI へのリクエストをキャッシュから返せます。

### GET でデータを絞り込む

絞り込み条件はクエリパラメーターで渡します。

```http
GET /users?role=admin&sort=name HTTP/1.1
GET /products?min_price=1000&max_price=5000 HTTP/1.1
GET /posts?tag=web&page=2&per_page=10 HTTP/1.1
```

GET にリクエストボディを含めることは仕様上可能ですが、
多くのサーバー実装が対応していないため避けるべきです。

---

## 4. POST

リソースを **作成** するためのメソッドです。
または、サーバーに何かを **処理させる** 目的でも使います。

```http
POST /users HTTP/1.1
Host: api.example.com
Content-Type: application/json

{"name": "Kazuki Matsuo", "email": "kazuki@example.com", "age": 21}
```

レスポンスで作成されたリソースの URI を返すのが一般的です。

```http
HTTP/1.1 201 Created
Location: /users/42
Content-Type: application/json

{"id": 42, "name": "Kazuki Matsuo", "email": "kazuki@example.com"}
```

### POST の特性

**非安全**: リソースを変更します（新規作成）。

**非冪等**: 同じ POST リクエストを2回送ると、2つのリソースが作成されます。
これが POST の重要な特徴です。

```
POST /orders （注文作成） を2回送る
→ 注文が2件作成される（意図しない場合がほとんど）
```

このため、フォームの二重送信防止が重要です。

### POST の使いどころ（GET・PUT・DELETE が使えない場合）

- リソースの新規作成
- 複雑な検索条件を持つ検索（ボディで条件を渡す）
- バッチ処理の実行
- ファイルアップロード

---

## 5. PUT

リソースを **置き換える（全更新）** メソッドです。
指定した URI のリソースを、リクエストボディで完全に上書きします。

```http
PUT /users/42 HTTP/1.1
Host: api.example.com
Content-Type: application/json

{"id": 42, "name": "Kazuki Matsuo", "email": "new@example.com", "age": 21}
```

### PUT の特性

**非安全**: リソースを変更します。

**冪等**: 同じ PUT リクエストを何度送っても結果は同じです。
（2回送っても同じ状態になるだけで、リソースが増えたりしない）

### PUT と POST の使い分け

| | URI | 冪等性 |
|-|-----|-------|
| POST | 新規作成（URI はサーバーが決める）| 非冪等 |
| PUT | 指定 URI のリソースを置き換える | 冪等 |

```
POST /users        → サーバーが /users/42 を生成して返す
PUT  /users/42     → クライアントが URI を指定して作成・置き換え
```

---

## 6. PATCH

リソースを **部分的に更新** するメソッドです。
PUT は全フィールドを送る必要がありますが、PATCH は変更する部分だけ送れます。

```http
PATCH /users/42 HTTP/1.1
Host: api.example.com
Content-Type: application/json

{"email": "new@example.com"}
```

これだけ送ると、`name` や `age` はそのままで `email` だけ更新されます。

### PUT vs PATCH の比較

```
ユーザー: {"id": 42, "name": "Kazuki", "email": "old@example.com", "age": 21}

PUT でメールアドレスを更新する場合:
→ {"id": 42, "name": "Kazuki", "email": "new@example.com", "age": 21}
（全フィールドを送る必要がある。省略したフィールドは null になる可能性）

PATCH でメールアドレスを更新する場合:
→ {"email": "new@example.com"}
（変更するフィールドだけ送る）
```

---

## 7. DELETE

リソースを **削除** するメソッドです。

```http
DELETE /users/42 HTTP/1.1
Host: api.example.com
Authorization: Bearer eyJ...
```

レスポンスは以下のどれかが一般的です。

```http
HTTP/1.1 204 No Content
（削除成功。ボディなし）

HTTP/1.1 200 OK
{"message": "削除しました"}
（削除成功。メッセージ付き）
```

### DELETE の特性

**非安全**: リソースを変更（削除）します。

**冪等**: 存在しないリソースを DELETE しても、結果は「リソースが存在しない状態」で同じです。
（1回目は削除、2回目以降は 404 を返すが、状態としては冪等）

---

## 8. HEAD

GET と同じリクエストですが、**ヘッダーだけ取得** してボディを返さないメソッドです。

```http
HEAD /files/large-video.mp4 HTTP/1.1
Host: cdn.example.com
```

```http
HTTP/1.1 200 OK
Content-Length: 1073741824
Content-Type: video/mp4
Last-Modified: Mon, 20 Apr 2026 12:00:00 GMT
```

用途：
- ファイルサイズを事前確認する（ダウンロード前に容量確認）
- リソースの存在確認（ボディを取得せずに済む分、高速）
- キャッシュの有効性確認（Last-Modified・ETag を確認）

---

## 9. OPTIONS

そのエンドポイントが **どの HTTP メソッドをサポートするか** を確認するメソッドです。

```http
OPTIONS /users/42 HTTP/1.1
Host: api.example.com
```

```http
HTTP/1.1 200 OK
Allow: GET, PUT, PATCH, DELETE, OPTIONS
```

特に **CORS（Cross-Origin Resource Sharing）** の「プリフライトリクエスト」として使われます。

---

## 10. 安全性と冪等性のまとめ

| メソッド | 安全（変更しない）| 冪等（何度送っても同じ）|
|---------|:-------:|:-------:|
| GET | ○ | ○ |
| HEAD | ○ | ○ |
| OPTIONS | ○ | ○ |
| POST | × | × |
| PUT | × | ○ |
| PATCH | × | △（実装次第）|
| DELETE | × | ○ |

### 冪等性が重要な理由

ネットワーク障害で「リクエストが届いたかわからない」状態になったとき、
冪等なメソッドであれば **安全に再送** できます。

```
GET・PUT・DELETE → 再送しても結果は同じ → 安全に再送可能
POST → 再送すると2回作成される → 注意が必要
```

---

## 11. CRUD と HTTP メソッドの対応

データベース操作の CRUD と HTTP メソッドの対応関係を覚えましょう。

| CRUD 操作 | HTTP メソッド | URI 例 |
|----------|-------------|-------|
| Create（作成）| POST | POST /users |
| Read（取得）| GET | GET /users/42 |
| Update（更新）| PUT / PATCH | PUT /users/42 |
| Delete（削除）| DELETE | DELETE /users/42 |

---

## まとめ

| メソッド | 使い方 | ポイント |
|---------|--------|---------|
| GET | リソース取得 | 安全・冪等・キャッシュ可 |
| POST | リソース作成 | 非冪等（二重送信注意）|
| PUT | リソース全置換 | 冪等。全フィールドを送る |
| PATCH | リソース部分更新 | 変更フィールドのみ送る |
| DELETE | リソース削除 | 冪等 |
| HEAD | ヘッダーのみ取得 | ボディ不要な確認に使う |
| OPTIONS | サポートメソッド確認 | CORS プリフライトに使う |
