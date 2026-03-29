---
name: auto-kanpu
description: デザインカンプJSON（座標・フォント・色・余白）とCSSを照合して自動修正
user-invocable: true
---

# auto-kanpu スキル

デザインカンプ（JSON）をもとにHTML/CSSの値を自動照合・修正するスキル

## 使い方
```
/auto-kanpu
```

---

## 🎯 目的

- デザインカンプのJSONとCSSの値を照合する
- フォントサイズ・余白・色・行間などのズレを自動修正する
- **CSSに個別ルールを増やさない**（既存の値を更新することを優先する）

---

## 🚀 Step 0. TodoWriteでチェックリスト表示（絶対厳守・スキル開始直後に必ず実行）

```
TodoWrite([
  { content: "デザインカンプ（JSON）をユーザーに要求する",        status: "in_progress" },
  { content: "HTML読み込み → クラス名とコンテンツを対応づける",  status: "pending" },
  { content: "CSS読み込み → 現在値を把握する",                   status: "pending" },
  { content: "JSONとCSSを照合 → ズレ一覧を作成する",             status: "pending" },
  { content: "バックグラウンドで自動修正",                       status: "pending" },
  { content: "残り問題をユーザーに報告する",                     status: "pending" }
])
```

---

## 📋 JSONフォーマット（受け取る形式）

ユーザーが貼り付けるJSONはこの形式：

```json
[
  {
    "label": "お 知 ら せ",
    "w": "36px",
    "h": "144px",
    "x": "1130.81px",
    "y": "784px",
    "textContent": "お\n知\nら\nせ",
    "textSize": "36px",
    "textFamily": "Yu Mincho",
    "letterSpacing": "0px",
    "lineHeight": "36",
    "color": "#000000",
    "opacity": "100%",
    "css": "top: 784px;left: 1131px;width: 36px;height: 144px;",
    "distanceToRight": "233.2px",
    "distanceToLeft": "1130.8px",
    "distanceToTop": "784.0px",
    "distanceToBottom": "-328.0px"
  }
]
```

---

## 📋 実行フロー

### Step 1. デザインカンプ（JSON）を要求する

ユーザーにJSONを貼り付けてもらう。

JSONが届いたら：
1. 各要素の `y` 座標と `textContent` でページ上の位置関係を把握する
2. ページ先頭（y=0）の要素を基準にして、各要素の相対距離を計算する

---

### Step 2. HTML読み込み

`Grep` で HTML の `class=` を抽出し、各クラスがどのJSONエントリに対応するかを把握する。

---

### Step 3. CSS読み込み + rem基準確認（絶対厳守）

#### 🚨 remの基準を必ず先に確認する

```
確認方法: Grep で html { font-size を検索する
```

| htmlのfont-size | 1rem = ? | px → rem 変換 |
|---|---|---|
| `62.5%` | 10px | px ÷ 10 |
| `calc(10 / 1400 * 100vw)` | 1400px幅で10px | px ÷ 10 |
| `100%`（未指定） | 16px | px ÷ 16 |

---

### Step 4. JSONとCSSを照合

#### 照合する項目

| JSON フィールド | CSS プロパティ |
|---|---|
| `textSize` | `font-size` |
| `textFamily` | `font-family` |
| `letterSpacing` | `letter-spacing` |
| `lineHeight` | `line-height` |
| `color` | `color` |
| `y`（要素間の距離） | `margin-top` / `padding-top` |
| `x`（要素間の距離） | `margin-left` / `padding-left` |
| `w`, `h` | `width`, `height` |

#### 座標から余白を計算する方法

```
要素Aのy + 要素AのH = 要素Aの下端
要素Bのy - 要素Aの下端 = AとBの間の余白
```

ただし、**ページ先頭の要素（メインビジュアルなど）の高さが現在のCSSと異なる場合**は補正が必要：

```
補正値 = 現在のページ先頭高さ - デザインカンプのページ先頭高さ
各要素の実際のY = JSONのy + 補正値
```

#### 照合表（例）

| クラス | CSS現在値 | JSON正解 | ズレ |
|---|---|---|---|
| .news_title | font-size: 3.0rem | 36px = 3.6rem | ✅修正対象 |
| .news_title | margin-top: 18rem | 184px = 18.4rem | ✅修正対象 |

---

### Step 5. 自動修正（CSSルール追加を最小限にする原則）

#### 🚨 CSS追記の原則（絶対厳守）

1. **既存の値を更新することを最優先**
   - 例：`margin-top: 18rem` → `margin-top: 18.4rem` に書き換える

2. **個別ルールの追加は最小限にする**
   - 2つの要素で値が違う場合：共通セレクタにデフォルト値を書き、差分だけ個別セレクタで上書き
   ```css
   /* ✅ 良い例 */
   #products1, #products2 { margin-top: 2.4rem; }  /* デフォルト */
   #products2 { margin-top: 11.9rem; }              /* 差分だけ上書き */

   /* ❌ 悪い例 */
   #products1 { margin-top: 2.4rem; }  /* 個別に追加 */
   #products2 { margin-top: 11.9rem; } /* 個別に追加 */
   ```

3. **font-family / color は上位クラスにまとめる**
   - 全体で同じ → `body` に1箇所
   - セクション内で同じ → 親クラスに1箇所
   - 個別バラバラ → 該当クラスに個別

#### 自動修正する項目

- `font-size` のズレ修正
- `font-weight` のズレ修正（Regular→400 / Bold→bold）
- `letter-spacing` のズレ修正
- `line-height` のズレ修正
- `color` のズレ修正
- `margin` / `padding` のズレ修正（座標計算から算出）
- AIコメント削除（`/* ○○px - JSON */` など）

---

### Step 6. 残り問題をユーザーに報告

```
## auto-kanpu 結果

### ✅ 自動修正済み（○件）
- .news_title: margin-top 18rem → 18.4rem
- .products_title: font-size 3.0rem → 3.6rem

### ⚠ 手動対応が必要なもの
- ...
```

---

## ⚠ 注意事項

1. **`@media` 内は修正しない**（SP用スタイルは対象外）
2. **Grepファースト**（全ファイルReadの前にGrepで絞る）
3. **個別ルールを増やさない**（既存セレクタへの更新を優先）
4. **座標は補正値を考慮する**（メインビジュアルの高さが違う場合）

---

## 🚨 座標のズレへの対応ルール（絶対厳守）

デザインカンプの座標が左右で微妙に違う（中央のはずがズレている）場合は、
**マージンで数値合わせしようとしない**。

### 優先順位

1. `text-align: center` → テキストや inline 要素のズレ
2. `margin: 0 auto` → ブロック要素を親の中央に配置
3. `justify-content: center` → flex の主軸方向の中央
4. `align-items: center` → flex の交差軸方向の中央

### ❌ やってはいけないこと

```css
/* ❌ 左右のズレをmarginで数値合わせしない */
.title { margin-left: 12px; }  /* 「なんか3pxずれてるから足す」はNG */
```

### ✅ やること

```css
/* ✅ 中央揃えのプロパティで解決する */
.title { text-align: center; }
.box   { margin: 0 auto; }
.flex-parent { justify-content: center; }
```

> デザインカンプ側の微妙なズレは意図的でないことが多い。
> 中央揃えで解決できるなら、そちらを優先する。
