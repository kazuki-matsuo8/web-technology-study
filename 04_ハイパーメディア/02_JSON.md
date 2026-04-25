# 第4章-02：JSON

---

## 1. JSON とは

JSON（JavaScript Object Notation）は、データを表現するための軽量なフォーマットです。

名前に「JavaScript」が入っていますが、言語に依存しない汎用的なフォーマットで、
現在は Web API のデータ交換形式として事実上の標準となっています。

```json
{
  "id": 42,
  "name": "松尾 一輝",
  "age": 21,
  "active": true,
  "email": null,
  "hobbies": ["プログラミング", "音楽"],
  "address": {
    "prefecture": "兵庫県",
    "city": "神戸市"
  }
}
```

---

## 2. JSON のデータ型

JSON は6種類のデータ型をサポートしています。

### 文字列（String）

ダブルクォートで囲みます。シングルクォートは使えません。

```json
"name": "Kazuki Matsuo"
"empty": ""
"unicode": "日本語も使えます"
```

エスケープが必要な文字：

| 文字 | エスケープ |
|------|-----------|
| `"` | `\"` |
| `\` | `\\` |
| 改行 | `\n` |
| タブ | `\t` |
| Unicode | `\uXXXX` |

### 数値（Number）

整数・小数・指数表記が使えます。
クォートは不要です。

```json
"age": 21
"price": 1299.99
"temperature": -5.5
"scientific": 1.5e10
```

注意：JSON に `Infinity` や `NaN` はありません。

### 真偽値（Boolean）

`true` または `false` です（小文字）。

```json
"active": true
"deleted": false
```

### null

値が存在しないことを表します。

```json
"nickname": null
```

### 配列（Array）

値の順序付きリストです。`[` と `]` で囲みます。

```json
"hobbies": ["プログラミング", "音楽", "読書"]
"numbers": [1, 2, 3, 4, 5]
"mixed": [1, "text", true, null]
"nested": [[1, 2], [3, 4]]
```

### オブジェクト（Object）

キーと値のペアの集合です。`{` と `}` で囲みます。
キーは必ずダブルクォートで囲んだ文字列です。

```json
{
  "key": "value",
  "nested": {
    "inner": "data"
  }
}
```

---

## 3. JSON の基本ルール

```json
{
  "name": "Kazuki",       ← キーはダブルクォート必須
  "age": 21,              ← 数値はクォート不要
  "active": true          ← 最後の要素にカンマは不要（trailing comma NG）
}
```

よくあるミス：

```json
// これはエラー（コメントも使えない）
{
  'name': 'Kazuki',         ← シングルクォートはダメ
  "age": 21,
  "active": true,           ← 最後にカンマ（trailing comma）はダメ
}
```

---

## 4. HTTP での JSON の使い方

### リクエスト（POST・PUT・PATCH でボディに含める）

```http
POST /users HTTP/1.1
Host: api.example.com
Content-Type: application/json

{"name": "松尾 一輝", "email": "kazuki@example.com", "age": 21}
```

`Content-Type: application/json` の指定が必要です。

### レスポンス

```http
HTTP/1.1 200 OK
Content-Type: application/json; charset=UTF-8

{
  "id": 42,
  "name": "松尾 一輝",
  "email": "kazuki@example.com"
}
```

---

## 5. JSON Schema

JSON データの構造を定義・検証するための仕様です。

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "id": {
      "type": "integer",
      "minimum": 1
    },
    "name": {
      "type": "string",
      "maxLength": 100
    },
    "email": {
      "type": "string",
      "format": "email"
    },
    "age": {
      "type": "integer",
      "minimum": 0,
      "maximum": 150
    }
  },
  "required": ["name", "email"]
}
```

API のリクエスト・レスポンスの形式を定義するために使われます。

---

## 6. JSON vs XML

かつては XML が Web API のデータ形式として主流でしたが、
現在は JSON が主流です。

| 比較 | JSON | XML |
|------|------|-----|
| 可読性 | 高い（シンプル）| やや低い（タグが多い）|
| ファイルサイズ | 小さい | 大きい（タグ分大きい）|
| パース速度 | 速い | 遅い |
| コメント | 使えない | 使える |
| 属性 | なし | 要素の属性が使える |
| 名前空間 | なし | あり |
| JavaScript との親和性 | 高い | 低い |

### 同じデータの比較

JSON:
```json
{
  "user": {
    "id": 42,
    "name": "Kazuki",
    "hobbies": ["coding", "music"]
  }
}
```

XML:
```xml
<user>
  <id>42</id>
  <name>Kazuki</name>
  <hobbies>
    <hobby>coding</hobby>
    <hobby>music</hobby>
  </hobbies>
</user>
```

JSON の方が明らかに短くシンプルです。

---

## 7. JSON の注意点

### 日付・時刻

JSON に日付型はありません。文字列として表現します。

```json
{
  "created_at": "2026-04-26T10:00:00+09:00",   ← ISO 8601 形式を推奨
  "date": "2026-04-26"
}
```

ISO 8601 形式（`YYYY-MM-DDTHH:mm:ss±hh:mm`）が国際標準です。

### 大きな整数

JavaScript の Number は64ビット浮動小数点数なので、
大きな整数（2^53 以上）を正確に表現できません。

```json
{
  "big_id": 9999999999999999    ← JavaScript では精度が落ちる可能性
  "big_id_string": "9999999999999999"   ← 文字列として扱うのが安全
}
```

### バイナリデータ

JSON は文字列しか扱えないため、バイナリデータ（画像等）は
Base64 エンコードして文字列として含めます。
ただしサイズが1.33倍になるため、大きなバイナリは URL で参照するほうが一般的です。

---

## 8. JSON の周辺技術

### JSON Lines（JSONL）

1行に1つの JSON オブジェクトを書く形式です。
大量データのストリーミングやログに適しています。

```jsonl
{"id": 1, "name": "Alice"}
{"id": 2, "name": "Bob"}
{"id": 3, "name": "Carol"}
```

### JSON:API

Web API の JSON レスポンス形式を標準化した仕様です。

```json
{
  "data": {
    "type": "users",
    "id": "42",
    "attributes": {
      "name": "Kazuki",
      "email": "kazuki@example.com"
    },
    "relationships": {
      "orders": {
        "links": {
          "related": "/users/42/orders"
        }
      }
    }
  }
}
```

---

## まとめ

- JSON は軽量なデータ交換フォーマット。現代の Web API の標準
- 6種類のデータ型：文字列・数値・真偽値・null・配列・オブジェクト
- キーは必ずダブルクォート。末尾のカンマ・コメントは不可
- HTTP では `Content-Type: application/json` を指定する
- 日付は ISO 8601 形式の文字列で表現する
- XML より軽量・シンプルで現在は JSON が主流
