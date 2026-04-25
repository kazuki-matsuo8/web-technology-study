# 第5章-01：REST とは何か

---

## 1. REST の定義

REST（Representational State Transfer）は、**Web のアーキテクチャスタイル** です。

2000年、Roy Fielding（ロイ・フィールディング）が博士論文
「Architectural Styles and the Design of Network-based Software Architectures」で提唱しました。

フィールディングは HTTP/1.0 と HTTP/1.1 の設計にも深く関わった人物で、
REST は「なぜ Web は成功したのか」を分析し、その設計原則をまとめたものです。

---

## 2. "Representational State Transfer" を読み解く

REST という名前を日本語に直訳すると「表現状態転送」です。

```
Representational ── リソースの「表現（Representation）」を
State            ── 「状態（State）」として
Transfer         ── 「転送（Transfer）」する
```

つまり：
- Web 上のリソース（情報）は「表現」として転送される
- クライアントはリソースの表現を受け取ることで、アプリケーションの状態が変化する

---

## 3. REST はプロトコルではない

重要な誤解を最初に正します。

**REST は HTTP のことではありません。**
**REST はアーキテクチャの「スタイル（設計思想）」です。**

REST に従った設計をする場合、HTTP を使うことが多いですが、
理論上は HTTP 以外のプロトコルでも REST を実現できます。

また、「HTTP を使っている = RESTful」ではありません。
HTTP を使っていても REST の原則に反した設計はいくらでもできます。

---

## 4. RESTful / RESTful API

REST の原則に従った設計を **RESTful** と言います。
REST の原則に従った Web API を **RESTful API** または **REST API** と言います。

ただし、「RESTful」という言葉は非常に幅広く使われており、
「HTTP を使った API」という意味で使われることも多いです。
（厳密には REST の6つの制約をすべて満たすものが真に RESTful です）

---

## 5. REST が解決しようとした問題

1990年代、インターネット上に様々なシステムが登場しましたが、
それぞれが独自のプロトコルや設計を持っていました。

Web がなぜ爆発的に普及したかを分析すると、
URI・HTTP・HTML という単純で統一されたインターフェースが理由でした。

フィールディングはこの Web の設計原則を抽出し、
「こういう設計にするとスケーラブルで相互運用可能なシステムが作れる」
という原則（= REST）として定義しました。

---

## 6. リソース中心の設計

REST の核心は **リソース（Resource）** という概念です。

### 従来のアプローチ（手続き中心）

```
/getUser?id=42
/createUser
/updateUserEmail?id=42&email=new@example.com
/deleteUser?id=42
```

操作（動詞）が URI に含まれています。

### REST のアプローチ（リソース中心）

```
GET    /users/42           → ユーザーを取得
POST   /users              → ユーザーを作成
PATCH  /users/42           → ユーザーを更新
DELETE /users/42           → ユーザーを削除
```

URI はリソース（名詞）を表し、操作（動詞）は HTTP メソッドで表します。

---

## 7. REST の重要な概念

### リソース（Resource）

URI で識別できる情報のこと。

```
/users/42     → 「ID が42のユーザー」というリソース
/orders       → 「注文一覧」というリソース
```

### 表現（Representation）

リソースを具体的な形式で表したもの。
同じリソースを JSON・HTML・XML など異なる形式で表現できます。

```
リソース：「ID が42のユーザー」

JSON 表現:
{"id": 42, "name": "Kazuki"}

HTML 表現:
<html><body><h1>Kazuki</h1></body></html>
```

### 統一インターフェース（Uniform Interface）

REST では、すべてのリソースに同じインターフェース（HTTP メソッド）でアクセスします。
特定のリソースのために専用のアクセス方法を作らない。

```
どんなリソースも: GET / POST / PUT / PATCH / DELETE でアクセス
```

---

## 8. フィールディングが REST を定義した背景

フィールディングが REST を提唱した2000年は、SOAP を使った Web サービスが盛んでした。

SOAP（Simple Object Access Protocol）は XML ベースのプロトコルで、
メソッド呼び出しを HTTP でトンネリングします。

```xml
<!-- SOAP リクエストの例 -->
POST /UserService HTTP/1.1
Content-Type: text/xml

<soap:Envelope>
  <soap:Body>
    <getUser>
      <id>42</id>
    </getUser>
  </soap:Body>
</soap:Envelope>
```

SOAP の問題点：
- 複雑・冗長
- HTTP のメソッドを活かせない（すべて POST になる）
- キャッシュが効かない
- 学習コストが高い

REST はこれに対して「Web の本来の設計を活かすべき」という主張でした。

---

## まとめ

- REST は Roy Fielding が2000年に博士論文で提唱したアーキテクチャスタイル
- REST = プロトコルではなく「設計の原則・思想」
- 核心はリソース中心の設計：URI でリソースを表し、HTTP メソッドで操作を表す
- RESTful = REST の原則に従った設計
- Web の設計哲学を API 設計にも適用したもの
