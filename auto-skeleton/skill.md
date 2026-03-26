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

### 0. TodoWriteでチェックリスト表示（絶対厳守）
スキル開始時に**必ず**以下のTodoWriteを実行し、チャット画面にチェックリストを表示する。
各ステップ完了ごとにstatusを `completed` に更新すること。

```
TodoWrite([
  { content: "HTMLからクラス名・構造を読み取る",       status: "in_progress", activeForm: "HTMLからクラス名・構造を読み取り中" },
  { content: "CSSスケルトン生成 → ファイルに追記",     status: "pending",     activeForm: "CSSスケルトン生成・追記中" },
  { content: "HTMLコメントをCSS基準で更新",            status: "pending",     activeForm: "HTMLコメントをCSS基準で更新中" }
])
```

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

## 📝 セクションの区切りコメント（絶対厳守）

CSSファイルの各セクションの先頭は**必ずこの形式**で書く：

```css
/* ┌─────────────────────────────────────────┐
   │ セクション名                        │
   └─────────────────────────────────────────┘ */
```

---

## 🏠 セクション親クラスの必須プロパティセット（絶対厳守）

`_area` など**セクションの一番外側のクラス**には、必ず以下を空でもセットで書く：

```css
.xxx_area {
  width: ;
  height: ;
  background-color: ;
  color: ;
}
```

- `color` は文字色。書かない場合も**空のまま残す**（後で埋める前提）
- `background-color` も同様に空でも残す

---

## 🔗 `<a>` タグのクラスも必ず生成する

HTMLに `<a class="xxx">` があれば、**必ずCSSルールセットを生成する**。

```css
/* リンク */
.news_link {
  color: ;
}
```

- `color: ;` を空で入れておく（リンク色は必ず設定が必要なため）
- `<a>` にクラスがない場合 → 親セレクタで `.xxx a { color: ; }` を生成する

---

## ⚠ 注意事項

1. **flex/gridのみ `display` を入れる**: それ以外のプロパティは空のまま（ただし上記セットは除く）
2. **既存CSSに追記する場合**: セクションコメントの直後に挿入
3. **重複クラスは生成しない**: 共通クラス（section_label等）が既にあればスキップ
4. **articleは「ブロック」と呼ぶ**: 「記事1ブロック」の形式で
5. **コメントはCSS基準**: HTML側を後から合わせる
6. **`└` は作成段階用**: 提出時に削除する前提
7. **コメントはセクションに合った言葉を使う**: 他セクションからのコピペ禁止。実際の要素の中身に合わせた名前をつける
