# auto-kanpu-size スキル

デザインカンプ（JSON）をもとにHTML/CSSのフォントサイズ・ウェイトを自動修正するスキル

## 使い方
```
/auto-kanpu-size
```

---

## 🎯 目的

- デザインカンプのJSON（テキスト一覧）とCSSの値を照合する
- フォントサイズ・ウェイトのズレをバックグラウンドで自動修正する
- 自動修正できない問題（`<br>`の混在など）をユーザーに報告する

---

## 🚀 Step 0. TodoWriteでチェックリスト表示（絶対厳守・スキル開始直後に必ず実行）

```
TodoWrite([
  { content: "デザインカンプ（JSON）をユーザーに要求する",        status: "in_progress", activeForm: "JSON要求中" },
  { content: "HTML読み込み → クラス名とコンテンツを対応づける",  status: "pending",     activeForm: "HTML読み込み中" },
  { content: "CSS読み込み → 現在値を把握する",                   status: "pending",     activeForm: "CSS読み込み中" },
  { content: "JSONとCSSを照合 → ズレ一覧を作成する",             status: "pending",     activeForm: "照合中" },
  { content: "バックグラウンドで自動修正（サイズ・ウェイト）",   status: "pending",     activeForm: "自動修正中" },
  { content: "残り問題をユーザーに報告する",                     status: "pending",     activeForm: "報告中" }
])
```

各ステップ完了ごとに `completed` に更新すること。

---

## 📋 実行フロー

### Step 1. デザインカンプ（JSON）を要求する

ユーザーに以下を貼り付けてもらう：

```
以下のJSON形式でデザインカンプのテキスト情報を貼り付けてください。

[
  { "text": "...", "fontSize": "○○px", "fontFamily": "...", "fontWeight": "Regular or Bold" },
  ...
]
```

JSONが届いたら Step 2 へ進む。

---

### Step 2. HTML読み込み

`Grep` で HTML の `class=` を抽出し、各クラスがどのテキストコンテンツに対応するか把握する。

**対応づけの例**:
```
.info_date_en → "2021 JUL 1(THU)-3(SAT)"（24px、Regular）
.info_location → "at PARK SIDE HALL"（40px、Bold）
.access_hall   → "PARK SIDE HALL"（28px、Bold）
```

---

### Step 3. CSS読み込み

Grep でクラスごとの `font-size` / `font-weight` を抽出する。

---

### Step 4. JSONとCSSを照合

| クラス | CSSの現在値 | JSONの正解 | ズレ |
|---|---|---|---|
| .info_date_en | 4.8rem / 300 | 24px / Regular | ✅修正対象 |
| .access_hall | 10rem / 400 | 28px / Bold | ✅修正対象 |

**照合ルール**:
- `font-size`: JSONのpx値 ÷ 10 = remの正解値（例: 24px → 2.4rem）
- `font-weight`: Regular → 400 / Bold → bold（または700）/ Light → 300
- HTMLで1つの`<p>`に異なるサイズが混在している場合は「報告対象」とする

---

### Step 5. バックグラウンドで自動修正

以下を自動修正する（確認不要・バックグラウンド実行）：

1. **font-size のズレ修正** → CSSの値をJSONのrem値に書き換える
2. **font-weight のズレ修正** → 400（Regular）/ bold（Bold）に書き換える
3. **`/* Regular - JSON */` などのAIコメント削除** → JSONと照合のために付けたコメントは提出前に全削除
   - 削除対象パターン: `/* Regular - JSON */` `/* Bold - JSON */` `/* Light - JSON */` `/* ○○px - JSON */`

**修正しないもの**:
- `font-size` コメント（`/* 24px - JSON */` など）がついていても、値が正しければそのまま
- `@media` 内のSP用スタイルは修正しない（PC側だけ修正する）

---

### Step 6. 残り問題をユーザーに報告

自動修正できなかった問題を以下の形式で報告する：

```
## auto-kanpu-size 結果

### ✅ 自動修正済み（○件）
- .info_date_en: 4.8rem → 2.4rem / 300 → 400
- .access_hall: 10rem → 2.8rem / 400 → bold
- コメント削除: /* Regular - JSON */ ×5件

### ⚠ 手動対応が必要なもの

#### 1. <br>でサイズが混在している箇所
同じ<p>の中に異なるfont-sizeのテキストが<br>で並んでいる。
CSSでは1つのクラスに1つのサイズしか当てられないため、自動修正不可。

- index.html:93 → `.info_date_en` の中に「24px（日付）」と「18px（時間）」が混在
  👉 対応: <p>を2つに分けて、それぞれ別クラスをつける
  例）<p class="info_date_en">日付</p><p class="info_time_en">時間</p>

#### 2. HTMLにあってJSONにないもの（デザインカンプに記載なし）
HTMLには存在するが、JSONに対応するテキストがない要素。
サイズ・ウェイトの正解がわからないため、自動修正できない。

- `.global_menu a`（TOP / INFORMATION / GALLERYなどのナビ）
  👉 対応: 自分でサイズを決めるか、デザイナーに確認する
- `.contact_msg`（「出品に関するお問い合わせ」ボタン）
  👉 対応: 同上
- `.btn_google_map`（GOOGLE MAPボタン）
  👉 対応: 同上

#### 3. JSONにあってHTMLに見つからないもの（HTML側に実装されていない）
JSONには記載があるが、HTMLで対応するクラスが見つからない要素。
デザインカンプには存在するが、まだHTMLに書かれていない可能性がある。

- "○○○○"（○○px / ○○）→ HTMLのどのクラスにも対応するテキストがない
  👉 対応: HTMLにそのテキストが書かれているか確認する。なければ実装漏れの可能性あり
```

---

## ⚠ 注意事項

1. **`@media` 内は修正しない**: SP用スタイルは別管理のため対象外
2. **コメント付きでも値が正しければそのまま**: 値だけ確認、コメントのせいで修正しない
3. **Grepファースト**: 全ファイルを Read する前に Grep で該当箇所を絞る
4. **HTMLにないテキストはスキップ**: JSONにあってもHTMLのクラスと対応しなければ報告するだけ
5. **自動修正後は必ず報告する**: 何を変えたか一覧でユーザーに伝える
