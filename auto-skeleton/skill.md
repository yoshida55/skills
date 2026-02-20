# auto-skeleton スキル

HTMLの構造を読み取り、CSSスケルトン（空ルールセット+コメント）を自動生成するスキル

## 使い方
```
/auto-skeleton
```

---

## 🎯 目的

- HTMLのクラス名に対応するCSSルールセットを空で生成
- flex/gridには `display` プロパティだけ入れる
- コメントで親子関係・配置方向を明示
- **作成段階用**（提出時に `└` コメントは削除する前提）

---

## 📐 コメントルール（絶対厳守）

### 1. flex/grid の親コメント
末尾に `【】` でレイアウト情報を記載。子要素の名前を `+` で列挙する。

```css
/* 記事1ブロック【flex 横2列: サムネイル + テキスト側】 */
.news_item { display: flex; }
```

**フォーマット**: `【{flex|grid} {横|縦}{N}{列|行}: {子1} + {子2}】`

### 2. `└` で階層表示（flex/gridの直接の子のみ）
- flex/gridの直接の子 ➡ `└` をつける
- **最大2つまで**（`└ └` が上限）
- flex/gridの子でない要素 ➡ `└` なし

```css
/* 記事1ブロック【flex 横2列: サムネイル + テキスト側】 */
.news_item { display: flex; }

/* └ サムネイル（左） */
.news_img { }

/* └ テキスト側（右） */
.news_body { }

/* └ └ カテゴリ+日付【flex 縦2行】 */
.news_meta { display: flex; }

/* カテゴリバッジ */
.news_category { }

/* 日付 */
.news_date { }

/* └ └ 詳細文 */
.news_text { }
```

### 3. 方向ラベル `（左）（右）`
- **横並び（flex-direction: row / grid横）の子** ➡ `（左）` `（右）` をつける
- **縦並びの子** ➡ つけない（上から下は自明）

### 4. 親と子のコメントは同じ言葉で対応させる
親: `【flex 横2列: サムネイル + テキスト側】`
子: `└ サムネイル（左）` / `└ テキスト側（右）`

➡ 親を見れば子が予測でき、子を見れば親がわかる

### 5. HTML/CSSのコメントは統一（CSS基準）
- CSSでコメントを確定 ➡ HTMLコメントもそれに合わせる
- HTMLを先に書いた場合でも、CSS確定後にHTMLを修正する

---

## 🎯 実行フロー

### 1. ユーザーがHTMLを提示（クラス名付き）
```html
<article class="news_item">
  <a href="#">
    <img src="..." class="news_img" />
  </a>
  <div class="news_body">
    <div class="news_meta">
      <span class="news_category"></span>
      <time class="news_date"></time>
    </div>
    <p class="news_text"></p>
  </div>
</article>
```

### 2. AIがCSSスケルトンを生成
```css
/* 記事1ブロック【flex 横2列: サムネイル + テキスト側】 */
.news_item {
  display: flex;
}

/* └ サムネイル（左） */
.news_img {
}

/* └ テキスト側（右） */
.news_body {
}

/* └ └ カテゴリ+日付【flex 縦2行】 */
.news_meta {
  display: flex;
  flex-direction: column;
}

/* カテゴリバッジ */
.news_category {
}

/* 日付 */
.news_date {
}

/* └ └ 詳細文 */
.news_text {
}
```

### 3. HTMLコメントもCSSに合わせて更新
```html
<!-- 記事1ブロック【flex 横2列: サムネイル + テキスト側】 -->
<article class="news_item">
  <a href="#">
    <img src="..." class="news_img" />
  </a>
  <!-- └ テキスト側（右） -->
  <div class="news_body">
    <!-- └ └ カテゴリ+日付【flex 縦2行】 -->
    <div class="news_meta">
      ...
    </div>
    <!-- └ └ 詳細文 -->
    <p class="news_text"></p>
  </div>
</article>
```

---

## 📋 `└` 判定基準

| 条件 | `└` つける？ |
|------|------------|
| flex/gridの直接の子 | ✅ つける |
| 通常フローの子 | ❌ つけない |
| 3階層目以降 | ❌ つけない（`└ └` が上限） |

---

## ⚠ 注意事項

1. **flex/gridのみ `display` を入れる**: それ以外のプロパティは空のまま
2. **既存CSSに追記する場合**: セクションコメントの直後に挿入
3. **重複クラスは生成しない**: 共通クラス（section_label等）が既にあればスキップ
4. **articleは「ブロック」と呼ぶ**: 「記事1ブロック」の形式で
5. **コメントはCSS基準**: HTML側を後から合わせる
6. **`└` は作成段階用**: 提出時に削除する前提
7. **コメントはセクションに合った言葉を使う**: 他セクションからのコピペ禁止。実際の要素の中身に合わせた名前をつける
