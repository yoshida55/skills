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

## 🚫 絶対禁止ルール（これを破ったら全部やり直し）

> **ズレ一覧をユーザーに提示して「修正して」と言われるまで、CSSを1行も書いてはいけない。**
> ❌ 照合途中でEditツールを使う → 禁止
> ❌ 「たぶんここがズレてる」と思っても先に直す → 禁止
> ✅ 全項目照合 → ズレ一覧提示 → ユーザーOK → 一括修正 → 正しい順序

---

## 🚀 Step 0. TodoWriteでチェックリスト表示（絶対厳守・スキル開始直後に必ず実行）

```
TodoWrite([
  { content: "デザインカンプ（JSON）を受け取る（既にあればスキップ）",      status: "in_progress", activeForm: "JSON受け取り中" },
  { content: "rem基準をユーザーに確認（自分でGrepしない）",                 status: "pending",     activeForm: "rem基準確認中" },
  { content: "対象セレクタをGrepで特定",                                   status: "pending",     activeForm: "セレクタ特定中" },
  { content: "全項目照合（font/color/spacing/余白/サイズ）→ ズレ一覧作成", status: "pending",     activeForm: "全項目照合中" },
  { content: "🚫 ズレ一覧をユーザーに提示・OKをもらう（修正はまだしない）", status: "pending",     activeForm: "ズレ一覧提示中" },
  { content: "一括修正（ユーザーOK後のみ実行）",                           status: "pending",     activeForm: "一括修正中" },
  { content: "残り問題をユーザーに報告",                                   status: "pending",     activeForm: "残り報告中" }
])
```

### ⚡ ツールを最初に並列取得する（時短）
スキル開始時に `Grep` `Edit` `Read` `Glob` を並列でToolSearchして取得する。
1つずつ取得すると待ち時間が増えるため、まとめて呼ぶこと。

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

### Step 3. CSS読み込み + rem基準確認

#### ⚡ rem基準の確認方法（最速手順）

**まずユーザーに1行聞く（自分でGrepしない）：**

> 「`html { font-size` の行を貼ってもらえますか？（style.css か reset.css にあります）」

ユーザーが貼った行を見て判断する：

| htmlのfont-size | 1rem = ? | px → rem 変換 |
|---|---|---|
| `62.5%` | 10px | px ÷ 10 |
| `calc(10 / 1400 * 100vw)` | 1400px幅で10px | px ÷ 10 |
| `100%`（未指定） | 16px | px ÷ 16 |

#### ❌ やってはいけないこと
- 自分で `work.css` → 「見つからない」 → `style.css` → ... と複数ファイルを探し回る
- ユーザーに聞けばすぐ解決するのに、Grepで時間をかける

#### ✅ ユーザーが既にどこかに貼っていたら
会話の中に `font-size: calc(...)` や `font-size: 62.5%` が出ていればそれを使う（改めて聞かない）

---

### Step 4. JSONとCSSを照合

> ## 🚨 照合チェックリスト（絶対厳守・セクションごとに必ず全項目確認）
> TodoWriteで以下を出してからチェックを始める。1項目でも飛ばしたら全部やり直し。
>
> ```
> TodoWrite([
>   { content: "font-size 確認（textSize）",         status: "pending", activeForm: "font-size 確認中" },
>   { content: "font-family 確認（textFamily）",      status: "pending", activeForm: "font-family 確認中" },
>   { content: "letter-spacing 確認（letterSpacing）",status: "pending", activeForm: "letter-spacing 確認中" },
>   { content: "line-height 確認（lineHeight）",       status: "pending", activeForm: "line-height 確認中" },
>   { content: "color 確認（color）",                  status: "pending", activeForm: "color 確認中" },
>   { content: "background-color 確認（shapeの色）",   status: "pending", activeForm: "background-color 確認中" },
>   { content: "width / height 確認（w, h）",          status: "pending", activeForm: "width/height 確認中" },
>   { content: "margin / padding 確認（x, y座標）",    status: "pending", activeForm: "余白 確認中" },
>   { content: "ズレ一覧をユーザーに報告",              status: "pending", activeForm: "ズレ一覧 報告中" }
> ])
> ```

#### 照合する項目

| JSON フィールド | CSS プロパティ |
|---|---|
| `textSize` | `font-size` |
| `textFamily` | `font-family` |
| `letterSpacing` | `letter-spacing` |
| `lineHeight` | `line-height` |（**必ず確認。bodyのline-heightが上書きされる**）|
| `color` | `color` |
| `y`（要素間の距離） | `margin-top` / `padding-top` |
| `x`（要素間の距離） | `margin-left` / `padding-left` |
| `w`, `h` | `width`, `height` |（**必ずセットで確認。片方だけ修正しない**）|

#### 座標から余白を計算する方法

```
要素Aのy + 要素AのH = 要素Aの下端
要素Bのy - 要素Aの下端 = AとBの間の余白
```

#### 🚨 余白の基準点ルール（絶対厳守）

セクション間の余白を計算するとき、**タイトルではなく主要コンテンツ（画像など）の座標を基準にする**。

```
❌ タイトル（y=2042）を基準にして計算 → ズレる
✅ 画像（y=2138）を基準にして計算   → 正しい
```

手順：
1. 先に「どこからどこまでの間隔か」を目視で確認する
2. セクションの「一番目立つコンテンツ（画像・ボックス）」のyとhを使う
3. タイトルなど付属要素は基準にしない

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
- `line-height` のズレ修正（**計算方法: lineHeight ÷ textSize の数値で指定**）
  - 例: textSize=16px, lineHeight=16 → `line-height: 1`
  - 例: textSize=16px, lineHeight=24 → `line-height: 1.5`
  - ⚠ `body { line-height: 1.8 }` が全体に効くため、カンプと異なる場合は**必ず明示的に上書きする**
- `color` のズレ修正
- `margin` / `padding` のズレ修正（座標計算から算出）
- AIコメント削除（`/* ○○px - JSON */` など）

#### ✅ CSS修正後の自己チェック（修正のたびに必ず確認・TodoWrite不要）

CSSを1つ書いたら、適用前に以下を声に出して確認する：

```
① この値は親要素の高さ・幅に依存していないか？
   例: height: 100% → 親にheightが設定されていないと効かない / 子が崩れる

② 子要素に想定外の影響が出ないか？
   例: .item { height: 23rem } → 中の img が height:100% を使っていたら崩れる

③ 他のプロパティと矛盾していないか？
   例: display: flex と float の併用 / position: absolute と margin: auto の混在
```

NGがあれば修正してから報告する。OKなら適用してよい。

---

### Step 6. 修正チェックリストをユーザーに提示 → 一括修正

#### 🚨 絶対厳守：ユーザーに2回言わせない

照合が終わったら、**修正対象を全件チェックリスト形式で提示**してユーザーに確認してもらう。
ユーザーが「修正して」と言ったら一括で全部修正する。

```
## 修正チェックリスト（全○件）

### ヘッダー
- [ ] .header_area: background-color gray → white
- [ ] .header_area: padding-left 0 → 9.1rem
- [ ] .header_button: width 14rem → 20rem

### NEWSセクション
- [ ] .title: font-size 2.4rem → 3.6rem
- [ ] .news_date: font-size 1.2rem → 1.4rem
...

全部まとめて修正しますか？
```

#### ❌ やってはいけないこと
- 一部だけ修正してユーザーに残りを2回目で言わせる
- 「⚠ 手動確認が必要」として放置して、ユーザーが再度指摘するまで直さない
- 修正リストを出さずにいきなりファイルを書き換える

---

### Step 7. 残り問題をユーザーに報告

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
5. **縦書き（writing-mode）のときは迷わずユーザーに確認する**
   - 「視覚的にどっち方向が広すぎますか？」と1問聞く
   - 自力で方向を推測して計算しない（時間の無駄になる）

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
