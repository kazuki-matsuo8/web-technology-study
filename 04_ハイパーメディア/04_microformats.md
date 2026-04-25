# 第4章-04：microformats

---

## 1. microformats とは

microformats（マイクロフォーマット）は、既存の HTML に
**セマンティックな（意味のある）情報を付加する** ための慣習・規約です。

HTML を大きく変えずに、人間にも機械（検索エンジン等）にも
読める形でデータを埋め込む仕組みです。

```html
<!-- microformats なし -->
<div>
  <span>松尾 一輝</span>
  <a href="mailto:kazuki@example.com">kazuki@example.com</a>
</div>

<!-- microformats あり（hCard 形式）-->
<div class="vcard">
  <span class="fn">松尾 一輝</span>
  <a class="email" href="mailto:kazuki@example.com">kazuki@example.com</a>
</div>
```

`class` 属性に特定のキーワードを使うことで、
「これは人物の名前」「これはメールアドレス」という意味をマシンが理解できます。

---

## 2. microformats の思想

### POSH（Plain Old Semantic HTML）

microformats の基本は「普通の HTML を正しく意味的に使う」ことです。
難しい技術なしに、既存の HTML の書き方を少し変えるだけで実現できます。

### 人間とマシンの両方に readable

- 人間：ブラウザが普通に表示するので、見た目は変わらない
- マシン：特定の class 名・rel 属性などを解析して構造化データを取得できる

---

## 3. 主な microformats

### hCard（人物・組織の情報）

vCard（電子名刺フォーマット）を HTML で表現したものです。

```html
<div class="vcard">
  <span class="fn n">
    <span class="given-name">一輝</span>
    <span class="family-name">松尾</span>
  </span>
  <div class="org">神戸電子専門学校</div>
  <a class="url" href="https://example.com/">https://example.com/</a>
  <a class="email" href="mailto:kazuki@example.com">kazuki@example.com</a>
  <div class="adr">
    <span class="region">兵庫県</span>
    <span class="locality">神戸市</span>
    <span class="country-name">日本</span>
  </div>
</div>
```

### hCalendar（イベント情報）

iCalendar フォーマットを HTML で表現したものです。

```html
<div class="vevent">
  <span class="summary">Web 技術勉強会</span>
  <span class="dtstart" title="2026-05-01T14:00:00+09:00">
    2026年5月1日 14:00
  </span>
  <span class="location">神戸電子専門学校</span>
</div>
```

### rel-nofollow

リンクの属性として、検索エンジンにリンクを追わないよう指示します。

```html
<a href="https://example.com/" rel="nofollow">外部リンク</a>
```

### rel-license

コンテンツのライセンス（著作権）を示します。

```html
<a rel="license" href="https://creativecommons.org/licenses/by/4.0/">
  Creative Commons BY 4.0
</a>
```

### rel-alternate

このリソースの別の表現（例：Atom フィード）へのリンクです。

```html
<link rel="alternate" type="application/atom+xml"
      title="RSS フィード" href="/feed.atom">
```

---

## 4. microformats2

microformats の次世代版（microformats2）では、クラス名の命名規則が整理されました。

```
p-* ← プレーンテキストのプロパティ（p-name, p-summary など）
u-* ← URL のプロパティ（u-url, u-email など）
dt-* ← 日時のプロパティ（dt-start, dt-end など）
e-* ← 埋め込みマークアップのプロパティ（e-content など）
h-* ← ルートクラス（h-card, h-event, h-entry など）
```

```html
<div class="h-card">
  <span class="p-name">松尾 一輝</span>
  <a class="u-email" href="mailto:kazuki@example.com">kazuki@example.com</a>
  <time class="dt-bday" datetime="2004-12-17">2004年12月17日</time>
</div>
```

---

## 5. 構造化データの現在：JSON-LD と Schema.org

microformats の考え方は重要ですが、現在は **JSON-LD（JSON for Linked Data）** と
**Schema.org** の組み合わせが主流になっています。

HTML の `<script type="application/ld+json">` タグの中に
JSON でセマンティックデータを埋め込む方式です。

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Person",
  "name": "松尾 一輝",
  "email": "kazuki@example.com",
  "birthDate": "2004-12-17",
  "affiliation": {
    "@type": "Organization",
    "name": "神戸電子専門学校"
  }
}
</script>
```

Google 検索がこの情報を利用してリッチスニペット（検索結果に詳細情報を表示）を生成します。

---

## まとめ

- microformats は既存 HTML に意味情報を付加する慣習（class 属性を活用）
- 人間にもマシンにも読める「人機共読」のアプローチ
- hCard（人物）・hCalendar（イベント）・rel-nofollow などが代表例
- 現在は JSON-LD + Schema.org が主流
- アイデア自体（HTML でセマンティック情報を表現する）は現在も重要
