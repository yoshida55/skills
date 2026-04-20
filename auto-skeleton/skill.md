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
- flex/gridの直接の子 ➡ `└` をつける（1つだけ・2つ以上禁止）
- flex/gridの子でない要素 ➡ `└` なし

```css
/* 記事1ブロック【flex 横2列: サムネイル + テキスト側】 */
.news_item { display: flex; }

/* └ サムネイル（左） */
.news_img { }

/* └ テキスト側（右） */
.news_body { }

/* カテゴリバッジ */
.news_category { }

/* 日付 */
.news_date { }

/* 詳細文 */
.news_text { }
```

### 3. 方向ラベル `（左）（右）`
- **横並び（flex-direction: row / grid横）の子** ➡ `（左）` `（右）` をつける
- **縦並びの子** ➡ つけない（上から下は自明）

### 🚨 writing-mode: vertical-rl のときは方向が逆になる（絶対厳守）

親要素またはその祖先に `writing-mode: vertical-rl` がある場合、flex の方向は逆転する。

| writing-mode | flex-direction | 実際の並び方 | コメント |
|---|---|---|---|
| 通常 | row | 横並び | 【flex 横N列】 |
| 通常 | column | 縦並び | 【flex 縦N行】 |
| vertical-rl | row | **縦並び** | 【flex 縦N行】 |
| vertical-rl | column | **横並び** | 【flex 横N列】 |

```css
/* ✅ writing-mode: vertical-rl の親の中では column = 横並び */
/* ヘッダー内側【flex 横2列: ロゴ + ナビ】 */
.header_inner {
  display: flex;
  flex-direction: column; /* vertical-rl環境では横並び */
}
```

> CSSを書く前に、対象要素の祖先に `writing-mode` があるか必ず確認する。

### 4. 親と子のコメントは同じ言葉で対応させる
親: `【flex 横2列: サムネイル + テキスト側】`
子: `└ サムネイル（左）` / `└ テキスト側（右）`

➡ 親を見れば子が予測でき、子を見れば親がわかる

### 5. HTML/CSSのコメントは統一（CSS基準）
- CSSでコメントを確定 ➡ HTMLコメントもそれに合わせる
- HTMLを先に書いた場合でも、CSS確定後にHTMLを修正する

---

## 🎯 実行フロー

### 0. 書き先CSSファイルを確認（絶対厳守・最初に1行だけ聞く）

> 「書き先のCSSファイルはどれですか？（例: work.css / style.css）」

- 会話の中に既に出ていれば聞かない
- 答えをもらったら、以降そのファイルだけに書く

### 0b. TodoWriteでチェックリスト表示（絶対厳守）
スキル開始時に**必ず**以下のTodoWriteを実行し、チャット画面にチェックリストを表示する。
各ステップ完了ごとにstatusを `completed` に更新すること。

```
TodoWrite([
  { content: "書き先CSSファイルを確認",                status: "completed",   activeForm: "書き先CSS確認中" },
  { content: "HTMLからクラス名・構造を読み取る",       status: "in_progress", activeForm: "HTMLからクラス名・構造を読み取り中" },
  { content: "CSSスケルトン生成 + HTMLコメント更新（同時実行）", status: "pending", activeForm: "CSSスケルトン生成・HTMLコメント更新中" }
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

/* カテゴリバッジ */
.news_category {
}

/* 日付 */
.news_date {
}

/* 詳細文 */
.news_text {
}
```

### 2b. CSSとHTMLコメントは同時に書く（必ず並列実行）

> ⚡ CSSへの追記 と HTMLコメントの更新 は**独立した作業**なので、Editツールを同じメッセージで2つ同時に呼ぶ。
> 直列に書くと待ち時間が発生するため禁止。

```
// ✅ 正しい（同時）
Edit(CSSファイル, ...)  ← 同じメッセージに
Edit(HTMLファイル, ...)  ← 両方まとめて

// ❌ 禁止（直列）
Edit(CSSファイル, ...)
→ 完了を待ってから
Edit(HTMLファイル, ...)
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
    <div class="news_meta">
      ...
    </div>
    <p class="news_text"></p>
  </div>
</article>
```

---

## 📋 `└` 判定基準

| 条件 | `└` つける？ |
|------|------------|
| flex/gridの直接の子 | ✅ つける |
| それ以外 | ❌ つけない |

---

## 📝 セクションの区切りコメント（絶対厳守）

CSSファイルの各セクションの先頭は**必ずこの形式**で書く：

```css
/* ┌─────────────────────────────────────────┐
   │ セクション名                        │
   └─────────────────────────────────────────┘ */
```

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

## 🔗 auto-kanpu への引き継ぎ（スケルトン完了後に必ず確認）

スケルトン生成・HTMLコメント更新が完了したら、以下を確認する：

- **JSONが既に渡されている** → 確認なしでそのまま auto-kanpu を実行する
- **JSONがない** → 「デザインカンプのJSONはありますか？」と1行だけ聞く

> JSONがあれば skeleton → kanpu が1ターンで完結する。

---

## ⚠ 注意事項

1. **flex/gridのみ `display` を入れる**: それ以外のプロパティは空のまま（ただし上記セットは除く）
2. **既存CSSに追記する場合**: セクションコメントの直後に挿入
3. **重複クラスは生成しない**: 共通クラス（section_label等）が既にあればスキップ
4. **articleは「ブロック」と呼ぶ**: 「記事1ブロック」の形式で
5. **コメントはCSS基準**: HTML側を後から合わせる
6. **`└` は作成段階用**: 提出時に削除する前提
7. **コメントはセクションに合った言葉を使う**: 他セクションからのコピペ禁止。実際の要素の中身に合わせた名前をつける
